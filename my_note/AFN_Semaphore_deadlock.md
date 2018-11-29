
#### Code 

```
static NSString * const R1_START = @"第一个请求开始";
static NSString * const R1_END = @"第一个请求完成";

- (void)viewDidLoad {
    [super viewDidLoad];

    dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
    
    [self requestOneWithSuccessBlock:^{
        dispatch_semaphore_signal(semaphore);
    }];
    
    dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
    
    NSLog(@"done");
    
}

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
```

#### Code Analysis

1. 创建信号量`semaphore`

2. 发起请求

3. 信号量等待，阻塞主线程的执行。主线程等待信号增加。

4. 请求的回调方法在主线程执行，所以等待主线程向下执行。然后释放信号。

由于这个请求并没有指定`completionQueue`,所以它的回调方法在主线程执行，然而此时主线程在等待`semaphore`,而`semaphore`的增加又需要主线程继续向下执行到回调方法才会执行到。从而造成死锁情况。

> `AFNetworking`在没有指定`completionQueue`的时候，回调方法是在主线程执行

#### 问题关键

事件回调中的信号释放和信号的等待在同一条串行队列中

#### 解决方式

不要将信号的等待和信号的释放放到同一条串行队列中

```
        AFHTTPSessionManager *sessionManager = [AFHTTPSessionManager manager];
        
        sessionManager.completionQueue = dispatch_get_global_queue(0, 0);
```

通过`AFHttpSessionManager`的`completionQueue`指定网络请求的回调方法的执行队列。这样，信号的释放放入到了子线程中。

主线程信号量会保持等待。而回调方法是在另一条线程执行的。不会受到主线程的影响。

请求回来之后，回调方法执行，信号量释放

主线程接收到信号量，解除阻塞。

#### 注意点

使用信号量，尽量避免在主线程，它会阻塞主线程，导致卡顿问题

当通过信号量控制网络请求时，一定避免信号的等待和释放在**同一条串行**队列中。会导致死锁问题。

