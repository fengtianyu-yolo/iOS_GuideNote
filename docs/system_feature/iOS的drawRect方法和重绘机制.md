
### `drawRect:`方法的作用

当自定义一个`UIView`的子类时，会默认提供`drawRect:`方法。

系统会通过这个方法将这个视图进行绘制出来。并且视图的重绘也是在这个方法中完成的。

当我们想要在这个`UIView`中进行一些自己的图形绘制操作的时候就在这个方法中进行实现。

这个方法由系统自动的调用。

##### 注意点

1. 若使用UIView绘图，只能在drawRect：方法中获取相应的contextRef并绘图。

2. 需要注意的是`UIImageView`不能重写`drawRect`方法实现自定义绘图。因为在苹果文档中指出，`UIImageView`是专门为了显示图片的空间，用了最优显示技术，禁止调用`drawRect`方法

### `drawRect:`的调用时机

这个方法调用的前提是这个视图有`frame`

1. 首先，在加载视图的时候会调用这个方法。调用顺序为`loadView` -> `viewDidLoad` -> `drawRect:`
    
    从方法名也可以理解。首先进行视图加载，然后视图加载完成。然后进行视图绘制。
2. 调用`setNeedsDisplay`方法之后出发视图重绘操作。会自动调用`drawRect`方法
3. 在调用`sizeThatFits`后会自动进行重绘。

### 和视图布局相关的方法

* `setNeedsLayout`,将视图标记为需要重新布局。
* `layoutIfNeeded`,这个方法会检测如果有标记，立即调用`layoutSubviews`
* `layoutSubviews`,对子视图重新布局

`layoutSubviews`方法的触发时机

1. `addSubviews`之后会触发
2. `view`的`frame`更改之后
3. 滑动`UIScrollView`

`layoutSubviews`将子视图布局，然后通过`drawRect`方法进行绘制出来



