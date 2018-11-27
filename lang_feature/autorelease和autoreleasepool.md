在`MRC`环境下对象的释放需要手动调用`[obj release]`方法来将对象的引用计数减1。调用`release`方法后该对象的引用计数立即减1

`[obj autorelease]`与`release`的不同是它不会立即将对象的引用计数减1，而是延迟该对象的释放。    

**那么它延迟到什么时候。**

`autorelease`对象不会立即进行`release`操作，当这个对象所在`autoreleasepool`进行销毁的时候，这个对象才会进行`release`操作。    

**那么，这个对象被添加到哪一个`autoreleasepool`,这个`autoreleasepool`什么时候释放。**

对象创建之后默认是添加到系统维护的`autoreleasepool`中。   

**那么系统维护的这个`autoreleasepool`是什么时候释放**

对于系统维护的`autoreleasepool`的释放时机就要了解到线程，runloop与`autoreleasepool`的关系。

每一个线程都可以创建`NSRunLoop`，但是子线程的RunLoop没有主动获取时是不会创建的。主线程的`NSRunLoop`是系统来创建并启动的。可以简单的把这个runloop看做是一个死循环，这个runloop不断的接收事件然后处理事件。

系统在RunLoop里注册两个观察者

第一个观察者监听Entry事件即即将进入RunLoop，它的回调会执行`_objc_autoreleasePoolPush()`创建自动释放池

另一个观察者监听两个事件，当监听到RunLoop即将进入休眠时BeforeWaiting事件时，调用`_objc_autoreleasePoolPop()`和`_objc_autoreleasePush()`方法。释放旧的自动释放池并创建新的自动释放池。当监听到RunLoop退出的Exit事件时，调用`_objc_autoreleasePoolPop()`释放这个自动释放池。

当自动释放池进行释放的时候，会对所有加入到该自动释放池中的对象进行一次release操作。

***

`stringWithFormat` 返回的是一个autorelease对象

#### case one

```
__weak id reference = nil;
- (void)viewDidLoad {
    [super viewDidLoad];
    NSString *str = [NSString stringWithFormat:@"a string object"];
    reference = str;
    NSLog(@"viewDidLoad - %@", reference);
}

- (void)viewWillAppear:(BOOL)animated {
    [super viewWillAppear:animated];
    NSLog(@"viewWillAppear - %@", reference);
}

- (void)viewDidAppear:(BOOL)animated {
    [super viewDidAppear:animated];
    NSLog(@"viewDidAppear - %@", reference);
}
```

##### 输出结果

```
viewDidLoad - a string object
viewWillAppear - a string object
viewDidAppear - (null)
```

##### 代码分析

`reference` 弱引用 `str`,不会对`str`的引用计数产生影响，用来查看`str`是否已经释放。

`[NSString stringWithFormat:@"a string object"]` 创建一个`autorelease`字符串对象，此时该对象的引用计数是1 

通过`str`引用该字符串对象，引用计数变为2 

`viewDidLoad`方法执行完毕，`str`指针释放，字符串对象的引用计数变为1

`autoreleasepool`drain的时候字符串对象进行`release`操作，引用计数变为0，销毁该对象


由输出结果可以看到，系统维护的autoreleasepool在`viewWillAppear`的时候还没有进行drain操作，在`viewDidAppear`的时候已经执行了drain操作，字符串对象被释放了。

这种情况是加入到系统默认的`autoreleasepool`中

#### case two

```
__weak id reference = nil;
- (void)viewDidLoad {
    [super viewDidLoad];
    @autoreleasepool {
        NSString *str = [NSString stringWithFormat:@"a string object"];
        reference = str;
    }
    NSLog(@"viewDidLoad - %@", reference);
}

- (void)viewWillAppear:(BOOL)animated {
    [super viewWillAppear:animated];
    NSLog(@"viewWillAppear - %@", reference);
}

- (void)viewDidAppear:(BOOL)animated {
    [super viewDidAppear:animated];
    NSLog(@"viewDidAppear - %@", reference);
}

```

##### 输出结果

```
viewDidLoad - (null)
viewWillAppear - (null)
viewDidAppear - (null)
```

##### 代码分析

`viewDidLoad`中`@autoreleasepool{}`即自己创建了一个自动释放池

当出了`@autoreleasepool{}`作用域，该`autoreleasepool`被drain，内部的字符串对象release。引用计数减一

离开了`@autoreleasepool{}`作用域，局部变量str被销毁，字符串对象引用计数减一，变为0。字符串被销毁。所以reference执行变为nil

#### case three

```
__weak id reference = nil;
- (void)viewDidLoad {
    [super viewDidLoad];
    NSString *str = nil;
    @autoreleasepool {
        str = [NSString stringWithFormat:@"a string object"];
        reference = str;
    }
    NSLog(@"viewDidLoad - %@", reference);
}

- (void)viewWillAppear:(BOOL)animated {
    [super viewWillAppear:animated];
    NSLog(@"viewWillAppear - %@", reference);
}

- (void)viewDidAppear:(BOOL)animated {
    [super viewDidAppear:animated];
    NSLog(@"viewDidAppear - %@", reference);
}
```

#### 输出结果

```
viewDidLoad - a string object
viewWillAppear - (null)
viewDidAppear - (null)
```

#### 代码分析

相比第二种情况，将`str`变量放到了`@autoreleasepool{}`的外边，所以在`@autoreleasepool{}`之后字符串对象虽然release了一次，但是`str`还在引用该对象。所以在`viewDidLoad`中并没有释放。

`viewDidLoad`之后，`str`变量销毁，没有对字符串的引用，则字符串对象被释放。

#### 什么时候主动使用autoreleasepool

```
for (int i = 0; i < 1000; i++) {
    NSString *var = [NSString stringWithFormat:@""];
}
```

当循环中产生大量的autorelease对象，这些对象在循环结束之后并不会立即释放。在上面这个例子中，这1000个字符串对象需要到这次runloop结束时才会被释放。造成短时间内存过高。

```
for (int i = 0; i < 1000; i++) {
    @autoreleasepool {
        NSString *var = [NSString stringWithFormat:@""];
    }
}
```

而使用了autoreleasepool之后，每次循环结束的时候autoreleasepool的作用域结束，这个对象就会立即被释放。避免造成了内存的过高。

#### 什么样的对象默认就是autorelease的

通常非`alloc`、`new`、`copy`、`mutableCopy`出来的对象都是autorelease的，比如`[UIImage imageNamed:]`、`[NSString stringWithFormat]`、`[NSMutableArray array]`等。

当不确定时可以通过case one 来检测是否该对象在作用域结束之后了立即释放来确定。


### 参考链接

* http://www.cocoachina.com/ios/20150610/12093.html