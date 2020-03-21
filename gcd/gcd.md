# GCD

## 重要名词概念

同步、异步、并发、串行

### 同步

同步操作的意思是当前操作会阻塞当前线程，直到这个任务完成之后线程再继续向下执行。

也就是当前线程会将这个任务提交到指定的队列中，然后开始等待，直到队列将这个任务出队的时候，仍然由当前线程执行这个任务。

任务执行完毕之后，线程继续向下执行指令。

```
	dispatch_sync(){}
```

1. 同步操作和线程之间的关系

    同步任务会仍然由当前线程去执行

### 异步是什么意思

```
	dispatch_async(){}
```

异步操作的意思是当前线程将block这个任务提交到指定队列之后就继续向下执行后面的操作，不用等待这个任务的完成。

### 串行队列

串行队列的特点是一定要等待上面的任务**执行完毕**之后才会执行下一个任务。所以串行队列的使用场景是当出现资源竞争的时候就需要使用串行队列来解决。

对于串行队列内核会开一条线程去执行串行队列中的任务

如果是同步提交到串行队列中的任务，当该任务被执行的时候，仍然由原来的线程执行

### 并发队列

而并发队列的特点是一旦上面的任务**开始执行**，则就可以执行下一个任务。

并发队列中和线程数没有必然关系。开多少条线程是由线程池来调度的。

### 什么叫做上下文切换

在多线程的执行过程中，线程切换的会带来上下文切换的开销。

上下文切换：CPU从一条线程切换到另一条线程时会把当前的指令位置、寄存器等信息存储到内存的一块区域，并把下一条线程的相关信息读取。这个过程就是上下文切换

## 线程之间通信方式 

**什么叫做线程间通信：怎么把方法从当前线程切换到其他线程去执行，或者7说怎么把数据从当前线程传给其他线程**

1. `performSelector:onThread`

2. `dispatch_async`

## 如何进行线程同步

*同步：协同步调，就是把顺序排好*

什么是线程同步：多条线程访问同一个数据进行写入的时候，如果不进行管理的话就导致数据写的错乱。线程同步就是让线程有顺序的访问资源。

线程同步的方式：

1. 加锁

2. 串行队列

3. 信号量机制

## 常见问题

* 同步一定是在当前线程执行吗

    是的

* 串行队列一定是只有一条线程去执行这个队列中的任务吗

    **不一定，如果是同步提交到串行队列的任务，还是原来的线程执行**

    如下示例代码所示 

    ```

    - (void)viewDidLoad {
        [super viewDidLoad];
        dispatch_queue_t queue = dispatch_queue_create("custom_queue", DISPATCH_QUEUE_SERIAL);
        
        dispatch_async(queue, ^{
            NSLog(@"异步向串行队列中添加的任务 %@", [NSThread currentThread]);
        });
        
        dispatch_sync(queue, ^{
            NSLog(@"同步向串行队列中添加的任务开始执行");
            sleep(3);
            NSLog(@"同步向串行队列中添加的任务执行完毕 %@", [NSThread currentThread]);
        });
        
        NSLog(@"主线程执行");

        dispatch_async(queue, ^{
            NSLog(@"异步向串行队列中添加的任务2 %@", [NSThread currentThread]);
        });
        
    }
    ```

    输出结果

    ```

    	>>> 异步向串行队列中添加的任务 <NSThread: 0x6000033caf00>{number = 3, name = (null)}

    	>>> 同步向串行队列中添加的任务开始执行

    	>>> 同步向串行队列中添加的任务务执行完毕 <NSThread: 0x6000033953c0>{number = 1, name = main}

    	>>> 主线程执行

        >>> 异步向串行队列中添加的任务2 <NSThread: 0x6000033caf00>{number = 3, name = (null)}

    ```

    可以看到如上结果所示，异步提交串行队列`queue`中的任务会只有一条线程执行。同步提交的由原来的线程执行

## GCD中API的使用和应用场景

### `dispatch_group`

### `dispatch_barrier`

