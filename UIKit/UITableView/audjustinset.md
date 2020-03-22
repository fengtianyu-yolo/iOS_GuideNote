
# 为什么第一个cell的布局不是从tableView的(0,0)点开始

该情况如图所示

tableView为红色背景

没有导航栏时

![tableview top blank](https://github.com/cocacola-ty/Images/blob/master/tableview_cell_top_black_example.png?raw=true)

当有导航栏的情况

![tableview navigationbar](https://github.com/cocacola-ty/Images/blob/master/tableView_navgationbar.png?raw=true)

这种情况是因为，`UIViewController`中有一个属性`automaticallyAdjustsScrollViewInsets`。从属性名可以看出来其作用，该属性决定是否自动的调整scrollview的inset。默认值为`YES`

也就是说，默认情况下`UIViewController`会自动调整scrollview的inset，以保证scrollView的内容不被导航栏、状态栏等遮挡。

所以会出现上图的情况，当有导航栏时cell是从导航栏的下方开始布局显示的。没有导航栏时是从状态栏的下方开始布局的。

所以如果想要让cell从tableView的(0,0)开始布局。将`automaticallyAdjustsScrollViewInsets`属性设置为`NO`

### iOS11之后的适配

当iOS11之后这个`automaticallyAdjustsScrollViewInsets`属性被废弃了。所以发现在iOS11系统以上的手机即使将该属性设置了`NO`还是会自动调整`inset`。

iOS11之后`UIScrollView`增加了两个新的属性`UIScrollViewContentInsetAdjustmentBehavior`和`adjustContentInset`。用来代替`automaticallyAdjustsScrollViewInsets`产生的效果。

`UIScrollViewContentInsetAdjustmentBehavior`是一个枚举值，定义了如何进行布局。

`adjustContentInset`属性的值决定了scrollview中的内容区域的显示位置。(作用其实和`contentInset`的效果一样)。这个属性的值是系统自动计算的。系统根据`UIScrollViewContentInsetAdjustmentBehavior`选择的类型计算出这个值的大小。

* `UIScrollViewContentInsetAdjustmentAutomatic` : 这个选项表示 在有导航栏的控制器中设置上下的`adjustContentInset`。保证上下不被遮挡。即使上下方向不可滚动。其他情况与下面的选项一样。

* `UIScrollViewContentInsetAdjustmentScrollableAxes` : 在可滚动方向上调整`adjustContentInset`，在不可滚动的方向上不做调整。

* `UIScrollViewContentInsetAdjustmentNever` : 这个选项表示不去调整scrollview的contentView。即内容区域就从scrollview的(0,0)开始显示。`adjustContentInset`就是0。

* `UIScrollViewContentInsetAdjustmentAlways` : 这个选项表示将scrollview的内容区域调整到安全区域以内，保证视图都不被遮挡。`adjustContentInset` = `safeAreaInset`

因为在iOS11之后有了“安全区域”的概念。安全区域是指不被遮挡的区域，也就是`tabbar`和`navigationBar`之间的区域。

*** 

#### `contentOffset`属性

是一个偏移量。单位是坐标。表示的是控件的顶点相对于滚动视图的顶点的偏移量。或者说是控件的顶点在滚动视图中的位置。
 
影响的是显示的位置。不会影响布局。

比如设置`contentOffset = CGPointMake(0,100)`。表示将内容区域的(0,100)位置移动到scrollview的(0,0)位置。从`contentView`的(0,100)位置开始显示

#### `contentInset` 属性

内容区域边界距离控件的边界的距离。影响的是内容的显示位置。默认各边间距都是是0

比如设置`contentInset = UIEdgeInsetsMake(10, 0, 0, 0)`，表示将内容区域顶部距离控件顶部有10个点的距离。
