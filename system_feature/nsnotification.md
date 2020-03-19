# 通知

## 通知中心 NSNotificationCenter

### 通知中心的作用

添加观察者

通知的发送

给通知找到对应的观察者去执行

### 怎么实现的



### 通知的常见问题

1. 通知中心是**同步**发送通知的。

	也就是说`postNotification`之后，要等到观察者执行完了响应方法之后，post操作才算执行完了，才会继续向下执行

2. object参数的作用

	`[[NSNotificationCenter defaultCenter] postNotificationName:@"n1" object:@2];`

	一般情况会传nil，则表示只监听通知名为指定name的，无论是哪个对象的。

	object参数可以理解为：该object的通知

	这个参数用于过滤通知，比如观察者只监听：对象2的通知名为n2的通知。

	从通知的存储方式可以很清晰看到查到过程，先查找通知名，如果有object参数，再去查找该object，最后得到响应方法。

3. 通知的响应在哪个线程执行

	在哪个线程进行post，就在哪个线程执行响应的方法。

4. 如何在主线程执行通知的响应

	1. 在通知的响应方法里主动的切到主线程

	2. 通过`–addObserverForName:object:queue：usingBlock:`添加观察者。通过queue参数指定处理的队列

	3. `NSMachPort`

5. 通知和代理的区别

	通知的耦合度更低，只管发送不用关心观察者的操作；通知是一对多

6. 对一个通知多次添加观察者

	通知的响应方法会多次执行

### 不足

如果想要进行异步的发送通知、在特定的时机发送通知，只使用通知中心就无法满足。

对于有更复杂条件的情况，就可以使用**通知队列**来完成

## 通知队列

通知队列会对通知进行管理，放进通知队列里的通知会按照先进先出的方式进行派发

通知队列会持有一个通知中心，当队列里的通知满足条件时，会将通知交给通知中心，由通知中心发出去

### 通知队列的基本使用方式

```

// 通知队列的使用
- (void)useNotificationQueue {
    
    // 添加监听
    [[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(notificationAction) name:@"nq" object:nil];
    
    // 创建一个通知
    NSNotification *notification = [NSNotification notificationWithName:@"nq" object:nil];
    
    // 拿到通知队列
    NSNotificationQueue *queue = [NSNotificationQueue defaultQueue];
    
    // 将通知加入队列，并通过参数设定通知发出的时机
    [queue enqueueNotification:notification postingStyle:NSPostASAP];
}

````

### 通知队列可以完成的功能

将通知加入队列的时候，可以通过`postingStyle`和`coalesceMask`指定该通知发送的一些条件

```
    [queue enqueueNotification:notification postingStyle:NSPostASAP coalesceMask:NSNotificationCoalescingOnName forModes:@[]];
```
 
1. `postingStyle` ： 决定什么时候将这个通知交给通知中心发出去
	
	`NSPostWhenIdle` : 在当前runloop空闲的时候再将通知发送出去

	`NSPostASAP` ： 当前runloop完成之后立即发送

	`NSPostNow` ： 立即发送。（同步）

	当使用`NSPostASAP`和`NSPostWhenIdle`时一定要注意，当前线程一定要有runloop存在。因为子线程默认是没有开启runloop的，在没有开启runloop的线程中使用，通知会发布出去，因为找不到合适的发出时机，线程执行完就结束。

2. `coalesceMask` ： 决定是否将相同的通知合并成一个发出去

	`NSNotificationNoCoalescing` ： 不合并

	`NSNotificationCoalescingOnName` ：根据通知的名字进行合并
 
	`NSNotificationCoalescingOnSender` ：根据通知的object进行合并

	这个参数只有在`postingStyle`不是`NSPostNow`的时候才会有效。只有当异步的时候才能够对所有未发出的通知进行合并处理。

