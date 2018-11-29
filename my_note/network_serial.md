1. 实现网络请求顺序执行的几种方案及优缺点比较

2. 网络请求顺序执行的具体实现

### 实现网络请求顺序执行的方案

1. 回调中发起下次请求

    * 优点：最简单

    * 缺点：会产生回调地狱的问题。回调套回调。

2. dispatch_group

    * `dispatch_group`的本质实现还是通过的信号量机制，所以优缺点与信号量方式基本是一样的。只是API更加方便一些。

3. 信号量

    * 优点：系统API即可完成，无需第三方支持，不会产生回调地狱问题，是通过调度线程完成的。
    * 缺点：当请求的回调与wait在同一串行队列的时候会发生死锁。

4. PromiseKit 

    * 优点：链式编程，代码可读性较高，本质和回调方式是一样的。
    * 缺点：需要导入PromiseKit第三方库

5. 待补充...

### 代码实现

#### 请求的发起如下

```
- (void) requestOneWithSuccessBlock:(void(^)(void))successBlock {
        AFHTTPSessionManager *sessionManager = [AFHTTPSessionManager manager];
        sessionManager.requestSerializer = [AFJSONRequestSerializer serializer];
        sessionManager.responseSerializer.acceptableContentTypes = [NSSet setWithObjects:@"application/json",@"application/zip", @"text/json", @"text/javascript", @"text/html", @"text/plain", nil];
    
        NSLog(@"%@",R1_START);
        [sessionManager GET:@"http://www.weather.com.cn/data/cityinfo/101190408.html" parameters:nil progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
            NSLog(@"%@",R1_END);
            
            if (successBlock) {
                successBlock();
            }
        } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
        }];
}

- (void) requestTwoWithBlock:(void(^)(void))successBlock{
    
    AFHTTPSessionManager *sessionManager = [AFHTTPSessionManager manager];
    sessionManager.requestSerializer = [AFJSONRequestSerializer serializer];
    sessionManager.responseSerializer.acceptableContentTypes = [NSSet setWithObjects:@"application/json",@"application/zip", @"text/json", @"text/javascript", @"text/html", @"text/plain", nil];
    
    NSLog(@"%@", R2_START);
    [sessionManager GET:@"http://wthrcdn.etouch.cn/weather_mini?city=%E5%8C%97%E4%BA%AC%E5%B8%82" parameters:nil progress:nil success:^(NSURLSessionDataTask * _Nonnull task, id  _Nullable responseObject) {
        
        NSLog(@"%@", R2_END);
        if (successBlock) {
            successBlock();
        }
    } failure:^(NSURLSessionDataTask * _Nullable task, NSError * _Nonnull error) {
    }];
}

```

***

#### 信号量的方式实现

##### CODE 

```
/*通过信号量的方式实现顺序执行*/
- (void)serialBySemaphore {
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        
        dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
        
        [self requestOneWithSuccessBlock:^{
            dispatch_semaphore_signal(semaphore);
        }];
        
        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
        
        [self requestTwoWithBlock:^{
        }];
    });
}

```
##### CODE ANALYSIS

使用信号量需要注意，`dispatch_semaphore_wait()`方法是会阻塞当前线程，如果没有接收到信号量，就一直阻塞当前线程的执行。

所以一定要注意网络请求的回调是否和`wait`在同一条串行队列中。如果在同一条串行队列则导致死锁情况。

串行队列的性质导致了只会有一条线程来这个队列取任务执行，并且一个任务执行完毕之后才会取下一个任务。当请求发起之后线程就会执行`wait`操作，而当请求回来之后，需要等待`wait`之后才可以执行。然而`wait`又需要回调方法中的`signal`操作才能继续向下执行。相互等待导致死锁发生。

这也就是为什么在方法的开始将线程切换到子线程，**`AFNetworking`的回调方法如果没有指定`completionQueue`则默认提交到在主队列，也就是在主线程执行**，而将信号量相关操作切换到子线程之后，阻塞的就是这条子线程，等到请求完成之后，主线程执行回调方法，释放信号量，这条子线程接收到信号量，继续向下执行，发起下一个请求。

信号量的三个方法介绍

`dispatch_semaphore_create(0)` 创建一个值为0信号量

`dispatch_semaphore_signal(semaphore)` 将信号量`semaphore`的值增加1

`dispatch_semaphore_wait(semaphore,time)`, 阻塞线程的执行，等待信号量`semaphore`，只有信号量的值大于0的时候才向下执行并对信号量值进行减一

***

#### `dispatch_group`方式实现

##### CODE ONE

```
-(void) serialByGroupWait {
    
    dispatch_async(dispatch_get_global_queue(0, 0), ^{
        dispatch_group_t group = dispatch_group_create();
        
        dispatch_group_enter(group);
        [self requestOneWithSuccessBlock:^{
            dispatch_group_leave(group);
        }];
        
        dispatch_group_wait(group, DISPATCH_TIME_FOREVER);
        
        dispatch_group_enter(group);
        [self requestTwoWithBlock:^{
            dispatch_group_leave(group);
        }];
        
        dispatch_group_notify(group, dispatch_get_global_queue(0, 0), ^{
            NSLog(@"all request  done!");
        });
    });
}
```

这种方式的实现方式的原理与信号量相似，只是API不同。

##### CODE TWO

```
- (void) serialByGroupNotify {
    
    dispatch_group_t group = dispatch_group_create();
    
    dispatch_group_enter(group);
    [self requestOneWithSuccessBlock:^{
        dispatch_group_leave(group);
    }];
    
    dispatch_group_notify(group, dispatch_get_global_queue(0, 0), ^{
        [self requestTwoWithBlock:^{
        }];
    });
}
```

这种方式的不同就是讲第二个请求放到了组内任务完成的通知方法中。

当group中的所有任务都完成了，会执行`notify`方法。本质其实也是在监听这个任务组中的信号量是否都已完成。

*** 

#### 回调中执行的方式

##### CODE 

```
- (void) serialByCallBack {
    
    [self requestOneWithSuccessBlock:^{
        
        [self requestTwoWithBlock:^{
        }];
        
    }];
}

```

***

### DEMO 

https://github.com/cocacola-ty/demos/tree/master/SerialNetRequest