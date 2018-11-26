
# 控制器的视图加载

## `loadView`方法的作用

创建`UIViewController`的视图

## 什么时候调用`loadView`这个方法

当第一次使用控制器的视图时，如果视图为nil，则调用这个方法创建控制器的视图

```
UIViewController *vc = [[UIViewController alloc] init];
vc.view.backgroundColor = [UIColor redColor];
```
如上面代码所示，当`init`之后控制器的视图并没有创建或加载。

只有当`vc.view`调用视图的时候才会检查这个视图是否存在。不存在通过`loadView`去创建视图

## loadView如何加载视图

1. 如果控制器是通过`initWithNibName: bundle:`来创建的，那么`loadView`就会去加载指定的xib文件

2. 如果控制器是通过`init`方法创建，并没有指定xib文件。
    
    1. 首先去检查有没有和控制器同名的xib文件。有的话加载这个xib，没有的话执行第二步。

    2. 创建一个空白UIView

## 通过代码创建控制器的视图

如果想通过代码创建控制器的视图，重写控制器的`loadView`方法即可。

并且不需要调用`[super loadView]`,因为，当调用`[super loadView]`会默认创建一个空白视图，既然已经自定义视图了，默认创建的视图就不需要了。

# 控制器的生命周期

1. `init` - 初始化控制器

2. `loadView` - 当使用控制器的视图时就会通过`loadView`创建控制器的视图

3. `viewDidLoad` - 视图加载完成

4. `viewWillAppear` - 视图即将进行显示

5. `viewWillLayoutSubviews` - 视图即将要对子视图进行布局

6. `viewDidLayoutSubviews` - 视图完成了对子视图的布局

7. `viewDidAppear` - 视图完成显示

8. `viewWillDisappear` - 视图将要消失

9. `viewDidDisappear` - 视图完全消失