
# CALayer中的坐标系统

布局属性和坐标系统

## `frame` & `bounds` & `center`

`UIView`中的布局定位属性：`frame`、`bounds`、`center`

`CALayer`中与`UIView`对应的分别是:`frame`、`bounds`、`position`

`frame`是当前视图或图层在父级视图层级中的占据的空间和位置

`bounds`是内部坐标

`center`和`position`都是相对于父图层的`anchorPoint`的位置

**`UIView`的`frame`、`bounds`、`center`方法都是去改变的该视图的`layer`的`frame`、`bounds`、`position`。**

**`frame`的值是根据`bounds` `postion` `transform`计算出来的值,当任意一个属性更改时都会影响frame的值**

![](https://zsisme.gitbooks.io/ios-/content/chapter3/3.2.jpeg)

```
- (void)viewDidLoad {
    [super viewDidLoad];
    
    UIView *showView = [UIView new];
    showView.backgroundColor = [UIColor redColor];
    showView.frame = CGRectMake(100, 100, 200, 200);
    [self.view addSubview:showView];
    NSLog(@"默认的frame:%f,%f,%f,%f",showView.frame.origin.x, showView.frame.origin.y, showView.frame.size.width, showView.frame.size.height);
    
    showView.transform = CGAffineTransformMakeRotation(M_PI_4);
    
    NSLog(@"更改transform后的frame:%f,%f,%f,%f",showView.frame.origin.x, showView.frame.origin.y, showView.frame.size.width, showView.frame.size.height);
}
```

输出结果如下：

```
>>> 默认的frame:100.000000,100.000000,200.000000,200.000000
>>> 更改transform后的frame:58.578644,58.578644,282.842712,282.842712
```

## `anchorPoint` 锚点

`CALayer`会通过锚点`anchorPoint`和`position`来确定位置

`anchorPoint`是一个单位比例，默认是`{0.5, 0.5}`，也就是在图层的中心位置。`{0, 0}`表示图层的左上角位置

**当给`CALayer`设置了`position`和`anchorPoint`之后，图层就会将锚点的位置放到`position`的所在坐标。**

如：

1. 默认的`anchorPoint`是`{0.5, 0.5}`,给定`position`为`{50,50}`,表示将这个layer的中心放到父图层的`(50, 50)`位置

2. 设置`anchorPoint`是`{0, 0}`,设置`position`是`{50, 50}`, 表示将这个layer的左上角放到父图层的`(50, 50)`位置




