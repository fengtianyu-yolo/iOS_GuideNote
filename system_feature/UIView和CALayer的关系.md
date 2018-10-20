
### 问题的由来

对于常用的动画我们知道有`CAAnimation`和`UIView animateWithDuration:animations:`

对于`CAAnimation` 我们都知道是对`layer`进行操作的动画 

那么对于`UIView`的动画是对`layer`操作还是对`view`操作。

#### 执行的动画的时候是谁在进行动画 `UIView` or `CALayer` ?

1. `UIView` 没有进行动画
2. `CALayer`也是没有进行动画的。
3. `CALayer`的`presentLayer`在执行这个动画过程

##### 证明

```
- (void)viewDidLoad {
    [super viewDidLoad];
    UIView *myView = [UIView new];
    self.myView = myView;
    myView.backgroundColor = [UIColor redColor];
    myView.frame = CGRectMake(100, 100, 100, 100);
    [self.view addSubview:myView];
    
    NSLog(@"initial myView.layer.frame = %f, %f, %f, %f", myView.layer.frame.origin.x, myView.layer.frame.origin.y, myView.layer.frame.size.width, myView.layer.frame.size.height);
    NSLog(@"initial myView.frame = %f, %f, %f, %f", myView.frame.origin.x, myView.frame.origin.y, myView.frame.size.width, myView.frame.size.height);

    self.displayLink = [CADisplayLink displayLinkWithTarget:self selector:@selector(getRealTimeFrame)];
    [self.displayLink addToRunLoop:[NSRunLoop currentRunLoop] forMode:NSDefaultRunLoopMode];

    [UIView animateWithDuration:1.0 animations:^{
        self.myView.frame = CGRectMake(100, 200, 100, 100);
    } completion:^(BOOL finished) {
        [self.displayLink invalidate];
    }];
}

- (void) getRealTimeFrame {
    freshIndex += 1;

    NSLog(@"index = %d ;self.myView.layer.presentationLayer.frame = %f, %f, %f, %f", freshIndex , self.myView.layer.presentationLayer.frame.origin.x, self.myView.layer.presentationLayer.frame.origin.y, self.myView.layer.presentationLayer.frame.size.width, self.myView.layer.presentationLayer.frame.size.height);
    NSLog(@"index = %d ;self.myView.layer.frame = %f, %f, %f, %f", freshIndex , self.myView.layer.frame.origin.x, self.myView.layer.frame.origin.y, self.myView.layer.frame.size.width, self.myView.layer.frame.size.height);
    NSLog(@"index = %d ;self.myView.frame = %f, %f, %f, %f", freshIndex , self.myView.frame.origin.x, self.myView.frame.origin.y, self.myView.frame.size.width, self.myView.frame.size.height);
}
```

1. 创建一个自定义视图`myView`指定frame
2. 添加到当前控制器
3. 输出当前`myView`和`myView.layer`的初始状态的`frame`值
4. 创建一个`CADisplayLink`，每次调用调用自定义方法`getRealTimeFrame`
5. `getRealTimeFrame`方法输出当前时间点的`myView`、`myView.layer`、`myView.layer.presentLayer`的frame

##### 输出结果

`myView`的`frame`值是直接变化的，没有随着动画过程的渐变

`myView.layer`的`frame`也是直接变化的，也没有随着动画过程渐变。

`myView.layer.presentLayer`的`frame`的值是随着动画过程进行渐变的。

##### 证明二

```
    [UIView animateWithDuration:0.5 delay:0 options:UIViewAnimationOptionAllowUserInteraction animations:^{
        self.myView.frame = CGRectMake(100, 200, 100, 100);
    } completion:nil];
```

`UIView`动画中有一个选项是`UIViewAnimationOptionAllowUserInteraction`,即表示允许用户交互

设置动画过程中允许用户交互

为`UIView`添加点击事件，将动画时间设置一个比较大的时间，放慢动画过程

我们可以发现，当动画开始之后但未结束时。如下图

![animating](https://note.youdao.com/yws/public/resource/8013976dc46521ac2fc644c7bd2014c4/xmlnote/WEBRESOURCE38b2de2422f962edb3c66ee802a8e939/7623)

A 为动画开始之前的位置

B 为当前动画执行到的位置

C 为动画执行完毕之后的位置

在当前时间点，我们点击B的位置并没有响应事件。而点击C点的位置响应了事件。所以，对于`view`是直接就移动到了目标位置。

##### 结论

所以我们看到的动画过程其实是`layer.presentLayer`的变化过程。

对于`UIView`是直接就更改到指定位置了。


#### 那么对于`UIView`和`CALayer`之间是怎样的一种关系

##### `UIView`其实是不显示内容，都是layer来显示的。

可以理解为`UIView`只是一个容器，内容的显示是有`layer`来决定的。就像是电视机和屏幕的关系一样。

比如给`view`设置背景色`view.backgroundColor = [UIColor redColor]`，其实是给`view`的`layer`来设置的。

##### 采用下面来证明上面的结论

通过隐藏view.layer会发现View没有内容了。

```
view.layer.hidden = YES;
```

通过查看调用栈

1. `[MyView drawRect:]` 调用`drawRect`方法进行绘制

2. `[UIView(CALayerDelegate) drawLayer:inContext:]` `UIView`作为`layer`的代理，调用`drawLayer`方法

3. `CALayer drawInContext:]` 向下调用的是 `layer`的`drawInContext`绘制方法

4. `[layer display]` 最终`layer`调用`display`方法来显示出来。完成一个视图的显示过程

所以当对view进行显示设置时，view作为layer的代理去通知layer进行相应的更改并显示。view只是作为一个layer的代理者来将值传递给layer去处理。

##### `UIView`的`frame` 和 `CALayer`的`frame`之间怎样的关系

`UIView`是作为`layer`的代理，当改变`UIView`的`frame`的时候，`UIView`去改变`layer`的frame。

当获取view的frame的时候，view的getter方法内部去调用layer的frame的方法，最终仍然是返回layer的frame。

当直接去更改layer的frame，再去查看view的frame，会发现view的frame也会相应的改变。

所以可以说，对view设置frame之后，view就告诉layer设置frame。 layer设置好frame之后确定了显示的位置和大小，然后来进行显示。

__可以这样去理解，layer作为主体，layer确定了位置之后，view作为layer的附属，layer在哪view就只能在哪。__

#### 为什么可以进行动画

当直接改变view的frame等属性的时候如`view.frame = CGRectMake(0, 0, 100, 200)`，默认没有动画效果。

当时当如下方式更改view的frame等属性就会有动画效果

```
    [UIView animateWithDuration:1.0 animations:^{
        self.view.frame = CGRectMake(100, 100, 100, 100);
    }];
```

这是因为默认的view的rootLayer的隐式动画是关闭的，但是在`UIView`的`animationBlock`中，隐式动画被打开了，所以当更改属性值时会有动画效果

> view的rootLayer即创建view之后就有的layer，非主动添加的。`view.layer`返回的就是view的rootLayer

##### 什么是**隐式动画**。 

隐式动画，即对于一些属性，只要属性值改变了，这是个属性从当前值到目标值就会有一个动画改变的过程。自动就会有的。不需要其他额外操作。这个就是叫隐式动画。

##### 有哪些属性可以支持隐式动画

`layer`的属性的注释中带有`Animatable`的都可以。

![calayer](https://raw.githubusercontent.com/cocacola-ty/Images/master/calayer_snapview.jpg)

***

#### 最终结论，在iOS中的无论是什么方式进行的动画其实都是`layer`进行的动画。

***

### 扩展

#### `Apple`对于`UIView`和`CALayer`之间关系的一段说明

> Layers are not a replacement for your app’s views—that is, you cannot create a visual interface based solely on layer objects. Layers provide infrastructure for your views. Specifically, layers make it easier and more efficient to draw and animate the contents of views and maintain high frame rates while doing so. However, there are many things that layers do not do. Layers do not handle events, draw content, participate in the responder chain, or do many other things. For this reason, every app must still have one or more views to handle those kinds of interactions.

#### `CALayer`除了`presentLayer`之外还有什么

`CALayer`维护了三个`layer` 

* presentLayer
* modelLayer
* Render 

待补充....