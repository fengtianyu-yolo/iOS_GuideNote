
首先，只有`UIResponder`对象才能够处理事件。`UIView`、`UIViewController`、`UIApplication`等都是`UIResponder`的子类

当一个事件产生的时候，系统将该事件加入到由`UIApplication`管理的事件队列中。

然后`UIApplication`从事件队列中取出事件，将事件进行分发传递

#### 事件的传递过程

1. 产生事件之后，系统将该事件加入到由`UIApplication`管理的事件队列中

2. `UIApplication`从事件队列中取到事件，将事件交给应用程序的主窗口`keyWindow`

3. `keyWindow`内部会检查是否可以接收事件，然后返回接收该事件的下一个视图。处理流程如下：

    1. 检查自己是否可以接受事件。
    
    2. 调用`hitTest:withEvent:`方法，这个方法会返回一个最适合处理该事件的视图。该方法的内部的处理流程如下

        1. 调用`pointInside:withEvent:`方法，检查触摸点是否在自己的控件范围内。
        
            如果这个方法返回NO，说明当前触摸点不在控件的范围内，`hitTest:withEvent:`方法返回`nil`

        2. 如果当前点在自己的控件范围。在自己的子视图数组中倒序开始遍历子视图。调用子视图的`hitTest:withEvent:`方法。

            如果所有的子视图的`hitTest:withEvent:`方法都返回的是nil。那该方法就返回`self`。即当前控件就是最适合处理事件的视图。

            如果有子视图返回的是非空对象。那么就返回该子视图返回的对象。即该对象为最适合处理事件的对象。


##### 不能接受事件的情况

1. 设置控件不允许交互。`userInteractionEnable` = `NO`

2. 控件被隐藏

3. 透明度<0.01

#### 一句话说传递链

事件从上到下传递，给视图之后，就会调用`hitTest:withEvent:`方法，返回当前视图中适合处理事件的子视图。如果没有合适的子视图，返回self，当前视图来处理事件。

#### 应用场景

##### 扩大按钮的点击区域

重写按钮的`pointInside:withEvent:`方法。如果触摸点处于按钮周边20以内，按钮也认为处于自己的范围内。

```
- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event
{
    if (CGRectContainsPoint(CGRectInset(self.bounds, -20, -20), point)) {
        return YES;
    }
    return NO;
}
```

##### 子视图超出父视图范围时响应事件

由事件传递链可以知道，当子视图超出父视图的范围时，点击这一部分，首先是进入父视图的`hitTest:withEvent:`方法中去检查触摸点是否在自己的范围，这时会检查发现不在自己的空间范围。所以子视图是无法接收到这个事件的。

所以解决该问题的方法就是重写父视图的`pointInside:withEvent:`方法，将子视图超出父视图的这部分也加入父视图的响应范围。

```
- (BOOL)pointInside:(CGPoint)point withEvent:(UIEvent *)event
{
    BOOL flag = NO;

    // 遍历子视图，如果坐标处于子视图的frame范围内，返回YES，认为触摸点在当前控件范围内。
    for (UIView *view in self.subviews) {
        if (CGRectContainsPoint(view.frame, point)){
            flag = YES;
            break;
        }
    }

    return flag;
}
```

#### 事件的响应过程

当视图接收到事件之后，如果发现视图没有对事件进行响应处理(比如没有实现`touchesBegan:withEvent:`)，将事件交给上一级视图去处理响应。

比如`subsubView`和`subView`都是`UIView`则都可以处理点击事件比如`touchesBegan: withEvent:`，那么只有当`subsubView`不去处理点击事件的时候，才会有`subView`来处理。否则该事件由第一级的响应者来处理。

#### 总结

### 参考资料

https://www.jianshu.com/p/2e074db792ba

https://www.jianshu.com/p/c87de31b3985