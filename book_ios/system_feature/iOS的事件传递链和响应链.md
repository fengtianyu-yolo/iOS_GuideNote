
首先，只有`UIResponder`对象才能够处理事件。`UIView`、`UIViewController`、`UIApplication`等都是`UIResponder`的子类

当一个事件产生的时候，系统将该事件加入到由`UIApplication`管理的事件队列中。

然后`UIApplication`从事件队列中取出事件，将事件进行分发传递

#### 事件的传递过程

1. 首先，`UIApplication`接收到事件，将事件传递到应用程序的主窗口`KeyWindow`
    
2. `KeyWindow`接收到事件，
    1. 检查自己是否可以处理事件
    2. 通过`pointInside:withEvent:`判断触摸点是否在自己的区域
    3. 从后往前的遍历子控件，子控件进行上面的两部检查。
    4. 子控件满足1-2条件，将事件交给该子控件
    5. 该子控件继续以上面的条件向下查找
    6. 如果没有符合条件的子视图，则当前视图为最适合处理事件的视图
    
```
graph TD
UIApplication-->keyWindow
keyWindow-->rootVc.view
rootVc.view-->subView

```

#### 如何决定适合处理事件的子视图

事件传递给视图之后，就会调用`hitTest:withEvent:`方法，返回当前视图中适合处理事件的子视图。如果没有合适的子视图，返回self，当前视图来处理事件。

#### 事件的响应过程

当视图接收到事件之后，如果发现视图没有对事件进行响应处理(比如没有实现`touchesBegan:withEvent:`)，将事件交给上一级视图去处理响应。

```
graph RL
subsubView-->subView
subView-->rootVc.view
rootVc.view-->keyWindow
keyWindow-->UIApplication
```

比如`subsubView`和`subView`都是`UIView`则都可以处理点击事件比如`touchesBegan: withEvent:`，那么只有当`subsubView`不去处理点击事件的时候，才会有`subView`来处理。否则该事件由第一级的响应者来处理。

#### 总结

简单说，在