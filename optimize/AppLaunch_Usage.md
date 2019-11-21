# Instruments App Launch的用法

XCode11之后，Instrument提供了App Launch工具，可以用于查看App的启动过程，从而可以针对性的对启动速度进行优化

## 使用介绍

首先启动Instruments

![start](https://github.com/cocacola-ty/Images/blob/master/applaunch_start.png?raw=true)

然后选择App Launch

![choose](https://github.com/cocacola-ty/Images/blob/master/applaunch_choose.jpg?raw=true)

打开之后界面大概如下所示，点击左上角的红色按钮就会开始App的启动分析。

![launch](https://github.com/cocacola-ty/Images/blob/master/applaunch_init.png?raw=true)

当启动App之后会运行5秒，这个时间是默认的，可以自己进行更改。然后自动停止App，并开始进行分析，完成之后就会将结果显示出来。如下图所示

这里会显示一个启动的简要情况，右侧默认会显示该过程的CPU使用情况。下方会显示线程追踪情况：线程运行的时长、被阻塞的时长、中断的时长等。

![default](https://github.com/cocacola-ty/Images/blob/master/applaunch_analyize.png?raw=true)

这时，选择你的App，在process的位置会有展开符，单击之后可以选择显示的内容。选择Time Profile,右侧的显示会更改为各个阶段的耗时情况。

右侧会将启动过程划分为几个阶段：进程创建、系统初始化、运行时的初始化、UIKit初始化、启动、第一帧渲染。并分别显示各阶段的耗时情况。

下面会显示整个过程的详细信息。可以切换查看角度。分别有：Event:Context Switch Point (查看整个启动过程中上下文切换的位置) 、Profile(显示整个启动过程中的方法调用栈及方法耗时) 、App Life Cycle (启动过程中的各个阶段的耗时) 、Sample 

如下图

![](https://github.com/cocacola-ty/Images/blob/master/applaunch_process_display.png?raw=true)

一般来说，系统初始化阶段和UIKit初始化阶段做优化的空间较小。重点关注的阶段可能多在启动和第一帧渲染的阶段。

所以，可以通过三击某个阶段，来只查看该阶段的相关信息。如图，三击Launching阶段，下方的部分选择Profile，就可以只查看在Launching阶段，所有的方法调用栈和方法耗时情况

![](https://github.com/cocacola-ty/Images/blob/master/applaunch_lifedetail.png?raw=true)

除此之外，还可以查看在启动过程中的所有的线程情况。点击app前面的展开符号，会展开显示启动过程中App的所有线程，点击线程的展开符号可以选择显示线程的状态。

线程的状态会通过颜色进行标注。**灰色的表示当前线程被阻塞。红色的表示当前线程可以运行，有了CPU资源就可以立即运行。橙色的表示线程被中断，CPU去执行其他优先级更高的线程。蓝色的表示正在运行**

如下图所示。

![](https://github.com/cocacola-ty/Images/blob/master/applaunch_detail.png?raw=true)


