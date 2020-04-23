# UIViewController

## 控制器中的`loadView`方法

`loadView`方法的作用：创建`UIViewController`的视图

### 加载视图的过程

1. 如果控制器是通过`initWithNibName: bundle:`来创建的，那么`loadView`就会去加载指定的xib文件

2. 如果控制器是通过`init`方法创建，并没有指定xib文件。
    
    1. 首先去检查有没有和控制器同名的xib文件。有的话加载这个xib，没有的话执行第二步。

    2. 创建一个空白UIView

### 调用时机

获取控制器的视图的时候，如果控制器的视图是nil，则就会调用这个方法来进行加载

如下例所示

```
UIViewController *vc = [[UIViewController alloc] init];
vc.view.backgroundColor = [UIColor redColor];
```
如上面代码所示，当`init`之后控制器的视图并没有创建或加载。

只有当`vc.view`调用视图的时候才会检查这个视图是否存在。不存在通过`loadView`去创建视图

### 什么时候重写该方法

如果想要通过代码创建视图，重写该方法，就会通过该方法中的代码进行视图的创建，不会走默认的加载流程

**并且不需要调用`[super loadView]`**,因为，当调用`[super loadView]`会默认创建一个空白视图，既然已经自定义视图了，默认创建的视图就不需要了。

## 控制器的生命周期

1. `init` - 初始化控制器
2. `loadView` - 首次获取控制器的视图时创建控制器的视图
3. `viewDidLoad` - 视图加载完成
4. `viewWillAppear` - 视图准备进行显示
5. `viewWillLayoutSubviews` - 准备要对子视图进行布局
6. `viewDidLayoutSubviews` - 完成了对子视图的布局
7. `viewDidAppear` - 视图显示出来
8. `viewWillDisAppear` - 视图将要消失
9. `viewDidDisAppear` - 视图已经完全消失


## 常见问题

- `viewDidLoad`方法和`setter`方法的执行顺序

	开发时遇到过在`viweDidLoad`中用到了属性，但是属性未被赋值。是因为`setter`执行前先执行了`viewDidLoad`。

	出现这种情况是因为在进行赋值操作前，有通过`.view`获取视图的操作，所以先去加载了视图，然后执行的赋值。

- `init`和`dealloc`中是否可以使用setter

	不要使用setter。使用setter赋值当存在继承关系时可能会导致意外的错误。
	
	推荐通过实例变量来进行赋值，
	
	[具体分析见这里](/Base/UIKit/UIViewController/q1.md)

