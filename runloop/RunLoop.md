
# Runloop全面简介

## 什么是RunLoop, RunLoop的作用

### 什么是RunLoop

RunLoop就是一个运行的循环，一旦开始这个循环如果没有退出消息就会一直运行着。这个循环中进行接收消息和处理消息。

RunLoop实际是一个对象,这个对象提供了一个入口函数，线程执行了这个函数之后就会一直处于运行这个循环的过程中。

而在这个循环中则处理程序运行过程中的各种事件，比如触摸事件、定时器事件等。事件处理完之后就会等待下一次事件的发生。

### 为什么需要RunLoop

* 线程保活，使程序持续运行

    一般来讲，线程处理完一个任务之后就自动退出了。而对于应用来说，是一个持续的交互过程。所以就需要通过RunLoop这个死循环机制来保持主线程一直运行。

* 节省CPU资源，提高性能

    RunLoop在没有事件处理的时候会使线程进入休眠，从而节省CPU资源，提高程序性能。

## RunLoop在一个循环中是怎么做的，处理了哪些东西

1. RunLoop启动之前，会通知观察者，即将进入RunLoop。
2. 发送一个通知，告诉观察者，将要开始处理定时器到时的事件  ?? 这一步是开始定时器计时 还是开始处理定时器的事件
3. 通知观察者，准备要处理source0事件。(source0事件可以任务是用户产生的事件)
4. 开始处理定时器到时的回调事件和source0的事件，处理完之后执行下一步
5. 如果有系统事件处于等待处理状态，直接进入第9步，开始处理
6. 通知观察者，线程进入休眠状态
7. 线程进入休眠，直到下面的事件发生时
    * 定时器到时
    * 系统事件到达端口
    * RunLoop设置的时间超时
    * RunLoop被手动唤
8. 通知观察者，线程将要唤醒
9. 处理等待处理的事件
    * 如果用户定义的定时器到时了，处理定时器的回调事件。然后到第二步开始下一次循环
    * 如果系统发出事件了，处理系统的事件
10. 通知观察者，退出RunLoop

> [苹果文档](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html#//apple_ref/doc/uid/10000057i-CH16-SW23)

## RunLoop中有哪些东西 

### 观察者：CFRunLoopObserverRef

这个观察者用于监听RunLoop中状态的改变，可以监听到的RunLoop的状态有下面几种

```
    typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
        kCFRunLoopEntry = (1UL << 0),               // 即将进入Loop：1
        kCFRunLoopBeforeTimers = (1UL << 1),        // 即将处理Timer：2    
        kCFRunLoopBeforeSources = (1UL << 2),       // 即将处理Source：4
        kCFRunLoopBeforeWaiting = (1UL << 5),       // 即将进入休眠：32
        kCFRunLoopAfterWaiting = (1UL << 6),        // 即将从休眠中唤醒：64
        kCFRunLoopExit = (1UL << 7),                // 即将从Loop中退出：128
        kCFRunLoopAllActivities = 0x0FFFFFFFU       // 监听全部状态改变  
    }
```

示例，监听RunLoop的状态

```
    - (void)viewDidLoad {
        [super viewDidLoad];
        CFRunLoopObserverRef ob = CFRunLoopObserverCreateWithHandler(CFAllocatorGetDefault(), kCFRunLoopAllActivities, YES, 0, ^(CFRunLoopObserverRef observer, CFRunLoopActivity activity) {
            NSLog(@"%zd", activity);
        });
        CFRunLoopAddObserver(CFRunLoopGetCurrent(), ob, kCFRunLoopDefaultMode);
    }
```

### 事件源：CFRunLoopSourceRef

事件源，有两种事件类型source0和source1

source0的事件是APP内部事件，触摸、摇晃等

source1是系统内核的产生的事件

### 定时器: CFRunLoopTimeRef 

基于时间的触发器。(基本就是NSTimer)，当它加入到RunLoop时，RunLoop会注册对应的时间点，当时间点到时，RunLoop会被唤醒来执行那个回调

### mode：CFRunLoopModeRef

模式，系统默认定义了多种运行模式。RunLoop运行时都会有选择一个模式运行。

* `kCFRunLoopDefaultMode` : App的默认运行模式
* `UITrackingRunLoopMode` : 用户交互模式
* `UIInitialzationRunLoopMode` : 启动APP时进入的第一个Mode，完成启动后就不再使用
* `kCFRunLoopCommonModes` : 占位模式，或者叫做标记模式
* `GSEventReceiveRunLoopMode` : 接收系统内部事件，通常用不到这个。

当RunLoop从一种模式切换到另一种模式的时候，RunLoop会退出并选择模式后在重新进入

对于定时器来说，是被加入到指定模式下的RunLoop，所以当切换RunLoop的模式之后，不会保留上一种模式的定时器。通过下面的示例来进行解释。并同时解释关于`kCFRunLoopCommonModes`

```
- (void)viewDidLoad {
    [super viewDidLoad];
    [self.view addSubview:self.tableView];
    NSTimer *timer = [NSTimer timerWithTimeInterval:2 repeats:YES block:^(NSTimer * _Nonnull timer) {
        NSLog(@"timer run");
    }];
    [[NSRunLoop currentRunLoop] addTimer:timer forMode:NSDefaultRunLoopMode];
}
```
上面这个例子中，我们先添加一个tableView到视图中，然后添加了一个`timer`到RunLoop中。

当程序启动之后，不做任何操作，会每2秒输出`timer run`，当我们滑动tableView的时候，会发现`timer run`不输出了，也就是`NSTimer`不工作了。当停止滑动的时候，又开始继续输出。

原因：

当不做任何操作的时候，RunLoop处于`NSDefaultRunLoopMode`模式下，而我们添加的`timer`也是添加到了这个模式下

当滑动`tableView`时，RunLoop结束`NSDefaultRunLoopMode`模式，选择`UITrackingRunLoopMode`模式启动,而这个模式下并没有添加`timer`所以就没有了输出。

当停止滑动，RunLoop又切换回`NSDefaultRunLoopMode`所有又有了输出

**通过这个例子证明了 定时器是与运行模式关联的**

想要让定时器在有用户交互和没有用户交互的模式下都运行。就用到了`kCFRunLoopCommonModes`

`KCFRunLoopCommonModes`其实并不是一种真实的模式，而是一个标记。当选择`kCFRunLoopCommonModes`时表示，在有`CommonMode`标记的模式下运行。

而`NSDefaultRunLoopMode`和`UITrackingRunLoopMode`都是有这个标记的。

所以，上面的示例中。将最后一行代码改为`[[NSRunLoop currentRunLoop] addTimer:timer forMode:kCFRunLoopCommonModes]`就可以在滑动与静止时都进行输出了

## RunLoop和线程之间的关系

线程和RunLoop是一一对应的，对应关系保存在一个全局的Dictionary中。

线程刚创建的时候是没有RunLoop的，如果不主动去获取RunLoop，线程就一直不会有。所以子线程中，如果没有主动获取RunLoop，子线程执行完任务就退出了。

而线程中的RunLoop的创建发生在获取的时候，在线程中获取RunLoop，系统会去全局字典中查找如果获取不到，会给当前线程创建一个RunLoop返回。

主线程的RunLoop是在启动的时候系统创建的。

### 什么场景下需要开启子线程的Runloop

## 苹果基于RunLoop实现的功能

## RunLoop的使用场景

***

## 参考资料

* https://blog.ibireme.com/2015/05/18/runloop/

* https://www.jianshu.com/p/d260d18dd551