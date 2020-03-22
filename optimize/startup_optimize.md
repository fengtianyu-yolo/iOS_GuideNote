# 启动速度优化

做启动速度优化的基本操作流程

1. 确定启动结束的时机点

2. 统计为优化时的启动数据

2. 解析启动过程，并划分启动阶段

3. 定位启动过程的耗时点

4. 对耗时位置进行优化

## 完整的启动过程

![]()

## 测量启动各阶段的用时

### 统一稳定的测试环境

* 同一个测试方法，每次都使用埋点数据或都使用XCUITest数据

* 断网或者使用飞行模式 

* 同一机型和系统

* Release模式

* 非低电量

* 非过热情况

### 如何查看启动的用时

线下启动用时的测量有下面三种方法：

1. 如果是iOS13之后的设备，可以通过`Instruments` -> `App Launch`查看。这个工具会把整个启动过程中的各个方法都列出来。如下图所示

2. 如果不使用`Instruments`，可以分别查看Pre-Main和Main阶段耗时

	Pre-Main阶段的耗时，可以通过`XCode` -> `Edit Scheme` -> `Enviroment Variables` 添加 `DYLD_PRINT_STATISTICS`值为1来查看。

	Main阶段的耗时，可以通过记录时间戳，进行比较

	这种方法最为简单，经试验，数据和线上Metrics的数据较为接近

3. iOS13之后的设备可以通过XCUITest中的Performance的测试来查看。这种方法不能查看具体的每个方法的耗时情况。

线上数据的监控

1. 对于线上的数据，苹果提供了Metrics工具进行监控。在`Orgnzer`中可以看到

2. 如果是埋点的话，结束时间可以在首页的didAppear中，而开始时间如果在main函数会丢失调per-main阶段的用时，有下面两个方法可以让开始的时间点更精准

	1. 以进程创建的时间。通过`sysctl`获得进程创建的时间戳。(参考美团的启动优化)

	2. 以任意一个类的load方法作为开始时间。

#### 线下三种方法测出来的数据对比

待补充... 

## 如何查找启动的性能问题

1. 可以通过AppLaunch或Time Profiler查看方法调用耗时。
	
	不过由于Instrument的逻辑是按一定的时间间隔去取线程堆栈，然后统计方法出现的次数。数据会有一些误差，感觉会偏低一些

2. 可以通过Hook，objc_msgSend函数，统计方法的耗时

	因为在Objc中，所有方法调用都会转为`objc_msgSend`函数的调用，所以可以通过拦截这个函数来统计每个方法的耗时。

	**`messier`工具的原理就是通过对`objc_msgSend`函数做的拦截统计的方法耗时，并且会生成火焰图**

	[hook`objc_msgSend`的实现方式](/如何hook系统函数)

## 如何优化
 
### ✨Main阶段的优化✨

1. 减少操作，梳理`didFinishLaunching`方法中的内容

2. 异步化执行。对于必须执行的启动任务，可以异步的进行异步执行

	异步任务的执行，需要控制并发量，降低启动时的压力

3. 监控MainRunloop，对于必须在主线程执行的任务，通过监控MainRunloop的状态，在空闲时执行

4. 优化首页视图结构

5. 检查IO操作

### Pre-Main阶段的优化

从上面的启动流程可以看到，有操作空间的优化阶段有，动态库、setupObjc Initializer 

减少文件加载

#### 如何定位到哪些load方法耗时较长

`App Launch` -> `Static Runtime Initialzer`

### 启动任务的管理

## 参考内容及重点

* [美团的启动优化]

 * https://tech.meituan.com/2018/12/06/waimai-ios-optimizing-startup.html

 启动任务的管理：提供一个宏来标识为启动任务。宏的内容是：通过Clang的编译器函数，将启动任务在编译时写进Mach-O的__DATA段。实现启动任务的分布式管理。

 线上监控时，通过获取进程创建时间作为开始时间节点。

* WWDC2016、WWDC2017、WWDC2018

	苹果主要对DYLD进行优化，并提供了Metrics和AppLaunch工具

## 相关工具的使用

AppLaunch 

Messier

## Link

https://mp.weixin.qq.com/s/Kf3EbDIUuf0aWVT-UCEmbA
