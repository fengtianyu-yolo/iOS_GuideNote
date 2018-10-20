当self强引用了block时，再在block中调用self会引发循环引用问题。所以，为了避免这种情况，都会使用weak-strong来解除循环引用问题。

如下所示

```
__weak typeof(self) weak_self = self;

a_block = ^{
    __strong typeof(weak_self) strong_self = weak_self;
    strong_self.view = ...
};
```

##### 代码解释

weak变量`weak_self`与`self`指向相同地址

在block中调用`weak_self`则不会导致循环引用。因为`weak_self`是weak修饰，所以不会增加`self`的引用计数。只有`self`对`block`的单向引用。

将weak变量转为strong变量则是为了在block执行期间不会被释放。因为weak变量不会影响对象的引用计数，对象可以正常释放，当对象释放之后该指针就变为nil。而将weak变量转为strong变量，则strong变量会持有该对象，所以在block执行期间，这个对象不会被释放，当block执行完毕之后，strong变量被销毁，该对象的持有减少。正常释放。

##### 问题

这样的写法需要在所有block的地方添加这两行代码

```
__weak typeof(self) weak_self = self;

__strong typeof(weak_self) strong_self = weak_self;
```

### 优雅的写法

定义一个宏定义

```
#ifndef weakify
    #if DEBUG
        #if __has_feature(objc_arc)
            #define weakify(object) autoreleasepool{} __weak __typeof__(object) weak##_##object = object;
        #else
            #define weakify(object) autoreleasepool{} __block __typeof__(object) block##_##object = object;
        #endif
    #else
        #if __has_feature(objc_arc)
            #define weakify(object) try{} @finally{} {} __weak __typeof__(object) weak##_##object = object;
        #else
            #define weakify(object) try{} @finally{} {} __block __typeof__(object) block##_##object = object;
        #endif
    #endif
#endif


#ifndef strongify
    #if DEBUG
        #if __has_feature(objc_arc)
            #define strongify(object) autoreleasepool{} __typeof__(object) object = weak##_##object;
        #else
            #define strongify(object) autoreleasepool{} __typeof__(object) object = block##_##object;
        #endif
    #else
        #if __has_feature(objc_arc)
            #define strongify(object) try{} @finally{} __typeof__(object) object = weak##_##object;
        #else
            #define strongify(object) try{} @finally{} __typeof__(object) object = block##_##object;
        #endif
    #endif
#endif 
```

当使用的时候

```
@weakify(self)

blcok = ^{
    @strongify(self)
    self.view = ...
}
```

#### 代码解释

首先看这个宏定义

这个宏定义关键的是在于这个

```
#define weakify(object) autoreleasepool{} __weak __typeof__(object) weak##_##object = object;
```

`autoreleasepool{}` 这个是没有实质作用的，只是为了当使用这个宏的时候再宏开头可以添加一个`@`符号，让使用方式更像其他语言的语法糖一样。

然后是`__weak __typeof__(object) weak##_##object = object` , 后面部分中`##`是用于连接的

`weak##_`解析之后就是 `weak_`,然后再与`##object`连接，所以最终就是`weak_object`,object是宏中传递进来的参数。

所以当使用这个宏的时候

```
@weakify(self) 
```

其实就转换成了 

```
__weak __typeof__(self) weak_self = self;
```

所以调用的时候应该是调用`weak_self`


然后来看这个宏

```
#define strongify(object) autoreleasepool{} __typeof__(object) object = weak##_##object;
```

这个宏的不同的地方是在`__typeof__(object) object = weak##_##object`

当我们传入`self`的时候这里就变成了这样

```
__typeof__(self) self = weak_self
```

即定义一个`self`变量来指向和`weak_self`相同的地址

所以在下面使用的`self`其实是自己定义的这个变量`self`
