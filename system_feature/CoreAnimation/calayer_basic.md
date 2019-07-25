
https://zsisme.gitbooks.io/

# 认识CALayer

1. UIView 和 CALayer的关系

`UIView`和`CALayer`是一个平行的层级关系

每一个UIView的对象中都有一个CALayer的对象

UIView的作用是创建并且管理这个CALayer对象

真正在屏幕上显示的是这个CALayer，UIView只是用来接收一些事件，然后再去操作自己的CALayer进行显示的变化

**这种架构设计的目的是，让操作和显示进行职责分离。**

比如，在iOS平台和MacOS平台，显示是相同的，而两个平台的操作是不同的，iOS平台是触摸，MacOS是鼠标等操作。这样就可以都使用CALayer来进行显示，各自封装不同的操作。

2. 为什么要使用CALayer

`UIView`已经对`CALayer`进行了封装，并且提供了一部分方法来对`CALayer`进行操作。比如`UIView animation..`方法进行动画

`UIView`在方法的内部通过更改`CALayer`来实现动画等显示效果的变化

但是`UIView`并不能将`CALayer`的全部功能进行封装。比如下面的功能就只能通过`CALayer`来进行实现

* 阴影、圆角、边框

* 透明遮罩

* 非线性动画

* 3D变化

## 了解CALayer的寄宿图

`CALayer`有一个属性叫`contents`

通过`contents`这个属性，可以让`CALayer`实现显示图片

这个属性是id类型，理论上可以接收任何类型，但是实际上，只有赋值`CGImage`的对象时才会显示一个图片。

_为什么定义为id类型?_

这是因为满足在MacOS系统上同时兼容NSImage和CGImage类型。所以定义为id类型。所以可以理解为这个属性只能接受`NSImage`和`CGImage`的对象

当使用时会发现，这个属性赋值的类型应该是`CGImageRef`类型，即指向`CGImage`的指针

```
layer.contents = (__bridge id)image.CGImage
```

_为什么要通过`bridge`进行转换？_

因为`CGImageRef`是一个CoreFoundation对象，id是一个Cocoa对象。所以需要进行转换

### 寄宿图的属性

* `contentsGravity`

这个属性的效果和`UIView`的`contentMode`对图片的效果是一样。用于改变图片的缩放和对齐方式

* `contentScale`

这个属性并不是单纯的用于对图片的放大和缩小,它的作用是为了让图片去适应不同分辨率的屏幕。

当使用`UIImage`的时候，在Retina屏幕它就会去读取Retina版本的图片，而CGImage不会因为屏幕分辨率去主动拉伸图片

所以通过设置`contentScale`来根据屏幕的情况进行对应的缩放来调整显示

_如果发现设置了`contentScale`并没有生效，检查是否`contentGravity`属性已经设置了缩放效果_

```
	layer.contentScale = [UIScreen mainScreen].scale
```

* `maskToBounds`

默认情况下，即使内容已经超过了`CALayer`的边界，他也会绘制出来。

`maskToBounds`属性设置为`YES`,`CALayer`不会去显示边界之外的内容

* `contentsRect`

通过这个属性，可以只显示寄宿图的一部分区域出来。

默认`contentsRect`是`{0,0,1,1}`，即整个寄宿图都是可见的。

当设置`contentsRect`为`{0, 0, 0.5, 0.5}`时，寄宿图会只显示左上角的1/4大小图。

![https://zsisme.gitbooks.io/ios-/content/chapter2/2.6.png]()

通过这个属性可以实现

将多张图片整合到一张大图上一次性加载。相比多次加载小图片，带来的好处有：内存使用、载入时间、渲染性能

当需要显示大图上的一个小图时，通过`contentsRect`去显示对应位置的小图。

示例如下：

```
@interface ViewController ()
@property (nonatomic, weak) IBOutlet UIView *coneView;
@property (nonatomic, weak) IBOutlet UIView *shipView;
@property (nonatomic, weak) IBOutlet UIView *iglooView;
@property (nonatomic, weak) IBOutlet UIView *anchorView;
@end

@implementation ViewController

/*
* 将裁剪的图片添加到指定layer上面
* image : 整张的图片
* rect : 要裁剪的位置
* layer : 显示小图片的layer
*/
- (void)addSpriteImage:(UIImage *)image withContentRect:(CGRect)rect ￼toLayer:(CALayer *)layer //set image
{
  layer.contents = (__bridge id)image.CGImage;

  //scale contents to fit
  layer.contentsGravity = kCAGravityResizeAspect;

  //set contentsRect
  layer.contentsRect = rect;
}

- (void)viewDidLoad 
{
  [super viewDidLoad]; //load sprite sheet

  // 加载整张大图
  UIImage *image = [UIImage imageNamed:@"Sprites.png"];

  // 显示iglooView的layer上面显示图片的左上角位置的内容
  [self addSpriteImage:image withContentRect:CGRectMake(0, 0, 0.5, 0.5) toLayer:self.iglooView.layer];

  // 在coneView的layer上显示整张大图的右上角部分
  [self addSpriteImage:image withContentRect:CGRectMake(0.5, 0, 0.5, 0.5) toLayer:self.coneView.layer];

  [self addSpriteImage:image withContentRect:CGRectMake(0, 0.5, 0.5, 0.5) toLayer:self.anchorView.layer];

  [self addSpriteImage:image withContentRect:CGRectMake(0.5, 0.5, 0.5, 0.5) toLayer:self.shipView.layer];
}
@end
```

* `contentsCenter`

这个属性的效果和`UIImage`中的`resizableImageWithCapInsets:`效果相同

默认情况下，`contentsCenter`的值是`{0, 0, 1, 1}`,表示整张图片会均匀拉伸开。

## 自定义CALayer的寄宿图

除了给`contents`属性赋值让`CALayer`显示图片，还可以通过绘制让`CALayer`显示一个寄宿图

`UIView`通过实现`drawRect`方法来给`CALayer`创建一个寄宿图

`drawRect:`方法没有默认实现，大多数情况下`UIView`并不需要给`CALayer`设置一个寄宿图

**当`UIView`检测到`drawRect:`方法实现之后，就会通过`Core Graphics`绘制一个图片，赋值给`CALayer`的`contents`。绘制的图片的像素大小就是视图尺寸乘以`contentsScale`。**

所以，当不需要自定义显示一个寄宿图的时候，不要去实现`drawRect:`方法，它会造成CPU和内存的浪费。

**`UIView`如何通过`drawRect:`绘制的图片设置给`CALayer`的寄宿图？**

`CALayer`定义了一个协议`CALayerDelegate`,当一个对象成为`CALayer`的代理需要实现这个协议中的方法

`CALayer`在重绘的时候，会检查是否有代理实现了协议中的`(void)displayLayer:(CALayer *)layer`方法 

`CALayer`的代理，可以通过在这个代理方法中直接设置`CALayer`的`contents`属性

如果代理没有实现`-displayLayer:`方法，`CALayer`会尝试调用`-(void)drawLayer:(CALayer *)layer inContext:(CGContextRef)ctx`。

这个方法会创建一个合适尺寸的空寄宿图，和一个`Core Graphics`绘制环境

当实现`drawRect:`进行了自定义绘制的图片时，`UIView`作为`CALayer`的代理会在内部实现`CALayer`的相关协议，实现`CALayer`显示绘制的内容






