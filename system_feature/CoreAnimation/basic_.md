
1. UIView 和 CALayer的关系

`UIView和CALayer是一个平行的层级关系

每一个UIView的对象中都有一个CALayer的对象

UIView的作用是创建并且管理这个CALayer对象

真正在屏幕上显示的是这个CALayer，UIView只是用来接收一些事件，然后再去操作自己的CALayer进行显示的变化

**这种架构设计的目的是，让操作和显示进行职责分离。**

比如，在iOS平台和MacOS平台，显示是相同的，而两个平台的操作是不同的，iOS平台是触摸，MacOS是鼠标等操作。这样就可以都使用CALayer来进行显示，各自封装不同的操作。

2. 为什么要使用CALayer



