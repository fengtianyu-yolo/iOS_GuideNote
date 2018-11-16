# UIViewController中影响布局的一些属性

## UIViewController中的默认配置

创建一个展示视图，将展示视图的位置放置在控制器的(0,0)位置。效果如下图。

* 默认情况，没有导航栏，没有tabbar

![](https://github.com/cocacola-ty/Images/blob/master/default_vclayout_nonav_notab.png?raw=true)

* 有导航栏，没有tabbar

![](https://github.com/cocacola-ty/Images/blob/master/default_vclayout_hasnav_notab.png?raw=true)

* 有导航栏 有tabbar

![](https://github.com/cocacola-ty/Images/blob/master/default_vclayout_nav_tab.png?raw=true)

从上面图可以看到

1. 导航栏和tabbar及状态栏都不会影响`UIView`的布局。控制器视图的(0,0)点一直都是屏幕的左上角。即使被导航栏遮挡。

2. iOS7之后，导航栏和tabbar都是半透明的。

## 影响布局的属性

### `edgesForExtendedLayout`

这个属性决定的是当前控制器的视图的边界是否延伸到屏幕边界。

这个属性是一个枚举值。默认值`UIRectEdgeAll`。即表示控制器的视图的边界都和屏幕的边界重合。

* `UIRectEdgeNone`：
    
    这个选项表示 ，当导航栏存在时，控制器的视图不会延伸到屏幕边界，而是延伸到导航栏和状态栏下面。即控制器的视图的上边边界在导航栏下方，不会被导航栏遮挡。

    **当导航栏隐藏时，仍然是将上边界延伸到屏幕上边界。**
    
    **但是即使导航栏完全透明，视图的上边界仍然是在导航栏的下方**
    
    需要注意，仍然会延伸到tabbar下面，也就是会被tabbar遮挡。

`UIRectEdgeTop` : 表示将控制器的视图的上边界放到屏幕的上边界。

`UIRectEdgeBottom`

`UIRectEdgeLeft`

`UIRectEdgeRight`

### `extendedLayoutIncludesOpaqueBars`

这个属性决定是否状态栏会影响布局。

**当状态栏是不透明的时候**，页面的默认布局是不包含状态栏的。也就是在状态栏下方的。

当这个属性设置为`YES`就会将状态栏也加入布局，也就是从屏幕顶点开始布局。

默认值是NO，也就是如果状态栏是不透明的，是不会被状态栏遮挡的。

### `automaticallyAdjustsScrollViewInsets`

这个属性表示如果`UIScrollView`是当前控制器视图的子视图。会自动调整`UIScrollView`的inset,以保证内容区域不被导航栏等遮挡。

这个属性已经废弃了。不再由控制器来决定。而是在`UIScrollView`中增加了`UIScrollViewContentInsetAdjustmentBehavior`属性来决定如何调整`UIScrollView`的inset以保证内容区域的显示。

#### `edgesForExtendedLayout`和`automaticallyAdjustsScrollViewInsets`的区别

这个属性和`edgesForExtendedLayout`的区别在于，`edgesForExtendedLayout`调整的是控制器视图的边界。而`automaticallyAdjustsScrollViewInsets`调整的是scollView的inset。

如下所示。

当`edgesForExtendedLayout`为`UIRectEdgeNone`时。

tableView向上滑动，导航栏后面看不到tableView，因为控制器视图的上边界在导航栏下面。超过了视图边界，所以看不到。

![](https://github.com/cocacola-ty/Images/blob/master/rectnone.png?raw=true)

而当`edgesForExtendedLayout`为`UIRectEdgeAll`时，可以发现导航栏后面是可以看到tableView的，因为控制器view和tableView的上边界都是在屏幕上边界的，只是将tableView的内容区域向下移动到了导航栏下方。

所以，滑动的时候没有超出父视图的范围，所以可以看到。

![](https://github.com/cocacola-ty/Images/blob/master/rectall.png?raw=true)

## 总结

默认情况下，控制器的视图的原点是在屏幕顶点。

设置控制器视图不延伸到屏幕后，只有导航栏隐藏时会延伸到屏幕，即使导航栏时透明的也不会延伸到屏幕。

