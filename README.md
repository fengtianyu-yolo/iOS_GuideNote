
### iOS Guide

* #### 关键字和语言特性的问题

    * [synthesize关键字的作用](lang_feature/synthesize关键字的作用.md)

    * [string属性用copy和strong修饰符时的不同](/lang_feature/string属性用copy和strong修饰符时的不同.md)
    
    * [属性修饰符`strong`、`weak`、`assign`、`copy`的作用](/lang_feature/属性修饰符分别的作用.md)
    
    * [`__weak`和`__block`的作用](/lang_feature/属性修饰符分别的作用.md?id=__weak和__block的作用)

    * `__block`在ARC和MRC下的作用一样吗
    
    * [`nonatomic` 和 `atomic`的作用](/lang_feature/属性修饰符分别的作用.md?id=nonatomic和atomic的作用)

    * [`weak`的底层实现原理](/lang_feature/weak的底层实现原理.md)

    * [`weak`属性如何自动置`nil`](/lang_feature/weak的底层实现原理.md?id=当weak指针指向的对象释放时，weak指针如何自动置为nil的)

    * `strong`的实现原理

    * 如何定义私有属性和私有方法

    * [OC中类和结构体的区别](/lang_feature/OC中类和结构体的区别)

    * 为什么`Category`中不能添加属性

* #### Runloop

    * 什么是RunLoop，RunLoop的作用是什么

    * RunLoop在一个循环中是怎么做的，处理了哪些东西

    * RunLoop中有哪些东西

    * RunLoop和线程的关系

    * 系统基于RunLoop实现的功能

    * RunLoop的应用场景

        * 基于RunLoop的机制，优化tableView滑动流畅度
    
    * RunLoop间如何进行通信

    * 如何在RunLoop空闲的时候处理特定任务

* #### Block相关问题

    * [block的本质，以及clang对block的实现](/block/block的本质.md)

    * `__block`的实现原理，为什么`__block`修饰的变量值可以改变

    * block是否要用copy修饰符

    * 有哪几种类型的block

* #### Runtime

    * [消息转发机制](/runtime/消息转发机制.md)

* #### 多线程

    * 线程之间如何进行通信

    * 用过哪些锁，哪些锁的性能比较高

* #### 系统框架和系统机制问题

    * 动画相关

        * 执行动画的几种方式

        * 动画是如何渲染显示的，哪条线程执行的动画

        * [UIView和CALayer的关系](/system_feature/UIView和CALayer的关系)

    * [iOS事件响应链和事件传递机制](/system_feature/iOS的事件传递链和响应链.md)

    * [iOS的drawRect方法和重绘机制](/system_feature/iOS的drawRect方法和重绘机制.md)

    * 使用drawRect有什么影响

    * [iOS的渲染机制和卡顿原因](/system_feature/渲染机制和卡顿原因.md)

    * KVO观察者机制

        * [KVO的实现原理](/system_feature/KVO/KVO的实现原理.md)

        * [KVO使用注意点](/system_feature/KVO/KVO使用注意点.md)

        * [如何关闭KVO和手动触发KVO](/system_feature/KVO/如何关闭KVO和手动触发KVO.md)

        * [KVO的基本使用示例](/system_feature/KVO/KVO的基本使用.md)

    * [load方法和initalize方法](/system_feature/load方法和initalize方法.md)

    * 如何去设计一个通知中心 / 通知中心的实现

    * 为什么UI更新需要在主线程执行

    * `loadView`的作用

* #### 如何进行优化

    * [视图调试工具的使用](/optimize/视图调试工具的使用.md)

* #### 工程设计问题

    * [如何追踪crash，线上crash率的追踪]()

    * 面向对象的设计原则是什么

    * 什么是简单工厂模式、工厂模式、抽象工厂模式

    * iOS中装饰器模式如何实现

* #### 计算机基础问题

    * HTTPS通信过程

### 辅助工具的使用

* [LLDB命令使用]()

*  [XCode缓存清理目录]()

* [`project.pbxproj `的解析及脚本修改证书配置思路实现](/assist_tool/project_pbxproj文件解析.md)

* [xcodebuild命令进行打包]()

* [XCode10下如何切换编译系统](/assist_tool/change_xcode_build.md)

* [`xcodebuild`使用旧版本编译系统](/assist_tool/change_xcode_build.md)

* [多XCode版本时更改`xcodebuild`使用的XCode版本](/assist_tool/change_xcode_build.md)
