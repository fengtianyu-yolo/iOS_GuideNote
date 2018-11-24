# block的存储区域

block可以存放在三种区域，栈区、堆区、全局数据区

## 全局区的block

### 什么时候在全局区

下面几种情况的block会存储在全局区

1. 定义在全局范围内的block

2. block中没有使用到任何外部变量

如下所示

```
- (void)viewDidLoad {
    [super viewDidLoad];
    void (^blk)(void) = ^{
        NSLog(@"xxx");
    };
    NSLog(@"%@", blk);
}
```

输出如下

```
>>> <__NSGlobalBlock__: 0x10c193088>
```

### 在全局区的作用

block存放在全局区和全局变量的效果是一样的。可以在全局范围内使用

## 栈区block

默认情况下通过block语法**创建出来**的block都是在栈区。

这里需要注意的是创建出来的这个block对象没有被引用时是在栈区。

在ARC环境如果将block赋值给一个变量，会将block从栈区拷贝到堆区。

```
- (void)viewDidLoad {
    [super viewDidLoad];

    int a = 10;
    NSLog(@"%@", ^(void){
        NSLog(@"%d",a);
    });
}
```

输出

```
>>>  <__NSStackBlock__: 0x7ffeed1c49d8>
```

## 堆区block

### 为什么要将block放在堆区

因为栈区中的block随着所处的作用域结束，block也就被释放了。

想要在所处的作用域之外使用block，就需要将block复制到堆区。

### 什么情况下会将block复制到堆区

1. 调用Block的`copy`方法时

2. 将Block赋值给`__strong`修饰符的变量或属性

3. GCD中的API中Block作为参数，其他Block作为参数的Cocoa框架中方法名带有`usingBlock`的方法

4. Block作为函数返回值返回时。

ARC有效的时候，在上面的情况下，编译器会自动判断是否需要将block复制到堆区。当编译器认为需要将block移动到堆区时，会自动生成将block从栈区复制到堆区的代码。

当编译器不能自动将block复制到堆区时，**需要我们使用`copy`方法，主动将block复制到堆区。**

### 示例

```
- (void)viewDidLoad {
    [super viewDidLoad];
    int a = 10;
    void (^blk)(void) = ^{
        NSLog(@"%d", a);
    };
    NSLog(@"%@", blk);
}
```

因为默认情况下变量`blk`是`__strong`修饰的。所以`blk`变量对block进行了强引用。

这时会调用`objc_retainBlock(blk)`来进行持有。而`objc_retaainBlock()`其实就是`Block_copy()`函数。

`Block_copy()`函数的作用是将栈上的block赋值到堆上，并将地址返回给blk变量。

所以变量引用一个栈区block，这个block会自动复制到堆区。

当这个方法执行完毕，blk变量被释放，block对象没有了强引用，对象也就被释放。

**但是在`MRC`环境下，这个局部变量`blk`引用的block就不会复制到堆区。**

**所以，有一个常见的问题是`block`属性用`strong`还是`copy`修饰也是因此而来。**

在`MRC`时，由于局部变量引用的block不会复制到堆上，如果用strong修饰的属性指向这个block对象，当作用域结束之后，这个block就被释放了。仍然是无法在作用域之外使用这个block。

所以需要用`copy`修饰属性，在属性的`setter`方法中执行`copy`方法，将block复制到堆上，从而在作用域之外使用这个block。

而在`ARC`时，这个block一旦`__strong`变量被引用就会自动复制到堆上，所以属性即使是用`strong`修饰的，当block所在的作用域结束，由于这个block在堆上并且有属性引用这个block，所以这个block不会被释放。仍然可以在作用域之外使用。
