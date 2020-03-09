# 响应者链

## 事件响应过程

1. 通过事件传递链找到了最佳响应者

2. 检查该响应者是否处理了该事件。如果处理了，事件的响应过程完成。

3. 如果没有处理，则将事件传递给下一级响应者

4. 如果一直没有响应者处理，最终将事件丢掉

## 怎样认为响应者处理了事件

能够响应事件的都是`UIResponder`的子类对象，`UIResponder`提供了下面这四个方法处理事件

```

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event;

- (void)touchesMoved:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event;

- (void)touchesEnded:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event;

- (void)touchesCancelled:(NSSet<UITouch *> *)touches withEvent:(nullable UIEvent *)event;

```

当点击等触摸类事件发生后，响应者通过实现上面的方法实现事件处理

当响应者在响应方法中调用了super，则会将事件传递给下一级的响应者一起处理

## 不响应事件的情况

1. hidden = true

2. alpha = 0

3. userInteractive = NO 

4. 超出父视图

## 手势识别的触发时机

场景：父视图实现了`touchesBegan`、`touchesCancelled:`、`touchesEnded:`方法 、子视图添加了`UITapGestureRecognizer`手势识别

现象：父视图的`touchesBegan`、`touchesCancelled:`方法执行了，子视图的手势识别执行了

原因：

1. 当点击屏幕的事件传递到子视图时，首先由子视图响应这个事件，由于子视图没有实现`touchesBegan`方法，所以默认的向下传递，父视图的`touchesBegan`执行了

2. 父视图执行完之后，没有向下一级传递，所以`开始触摸`的这个事件就处理完了

3. `开始触摸`的事件处理完之后，继续回到最初的响应者，也就是子视图，处理下一个事件。

4. 这时候子视图的手势识别执行了，识别到了Tap事件。所以执行了Tap的响应方法。

5. 由于`开始触摸`这个事件被当做手势事件响应了，所以一开始的触摸事件被取消了。所以执行了`touchesCancelled`事件

6. 子视图没有实现`touchesCancelled`，所以下一级响应者响应，所以执行了父视图的`touchesCancelled`


如果父视图也没有实现`touchesCancelled`，这个事件最终就被丢弃掉。

所以Tap手势的识别是在touchesBegan之后识别到

如果子视图实现了touchesBegan方法并且没有调用super，那么这个触摸开始事件就直接由子视图处理了

### 代码如下

```

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    UIView *oneView = [UIView new];
    oneView.backgroundColor = [UIColor redColor];
    oneView.frame = CGRectMake(100, 100, 200, 200);
    [self.view addSubview:oneView];

    UITapGestureRecognizer *tapGes = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(tap)];
    [oneView addGestureRecognizer:tapGes];
}

- (void)tap {
    NSLog(@"tap response");
}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    NSLog(@"click vc view");
}

- (void)touchesEnded:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    NSLog(@"click vc view end");
}

- (void)touchesCancelled:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    NSLog(@"click cancel ... ");
}

```

输出：

```
>>> click vc view
>>> tap response
>>> click cancel ...
```

## 参考链接

https://segmentfault.com/a/1190000015060603