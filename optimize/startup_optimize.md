# 启动速度优化

## 完整的启动过程

## 测量启动各阶段的用时

### 如何让测试数据更准确

断网或者使用飞行模式 

### 如何查看启动的用时

1. 如果是iOS13之后的设备，可以通过`Instruments` -> `App Launch`查看。这个工具会把整个启动过程中的各个方法都列出来。如下图所示

2. 如果不使用`Instruments`，可以分别查看Pre-Main和Main阶段耗时

Pre-Main阶段的耗时，可以通过`XCode` -> `Edit Scheme` -> `Enviroment Variables` 添加 `DYLD_PRINT_STATISTICS`值为1来查看。

Main阶段的耗时，可以通过记录时间戳，进行比较

## 如何优化
 
### ✨Main阶段的优化✨

#### 优化启动项，减少`didFinishLaunching`中执行的操作

1. 首先，梳理在`didFinishLaunching`中的启动项，筛选出来可以延迟执行的启动项

2. 将可以延迟执行的启动项推迟执行
	
	在之前的优化过程中，我将延迟执行的启动任务全部放到了首页的`viewDidAppear`方法中，

	这样会带来一个问题，`viweDidAppear`也是主线程执行的，所以出现的情况就是启动速度会变快，但是显示首页之后，会卡住，不能进行事件响应。

	所以，需要还需要对延迟执行的启动任务进行管理

#### 解决耗时长的方法

1. 怎么查看方法的耗时

2. 怎么修改这些方法

#### 延迟执行的启动任务的管理

1. 子线程执行,

	一定要注意子线程的并发量及线程安全问题

	> 什么是线程安全

2. 必须在主线程执行的启动任务，如何处理

### Pre-Main阶段的优化

从上面的启动流程可以看到，有操作空间的优化阶段有，动态库、setupObjc Initializer 

减少文件加载

#### 如何定位到哪些load方法耗时较长

`App Launch` -> `Static Runtime Initialzer`

### Main阶段的优化

## 验证优化效果

## 相关工具的使用


## Link

https://mp.weixin.qq.com/s/Kf3EbDIUuf0aWVT-UCEmbA
