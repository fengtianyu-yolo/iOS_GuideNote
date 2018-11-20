# block的存储区域

block可以存放在三种区域，栈区、堆区、全局数据区

## 在全局区的block

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

## 什么时候block在栈区

默认情况下创建出来的block都是在栈区。

但是，在ARC环境如果将block赋值给一个变量，会将block从栈区拷贝到堆区。

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

## 什么时候在堆区

**ARC环境下**，如果变量引用一个栈区的block，这个block就会自动复制到堆区。如下所示

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

因为默认情况下变量是`__strong`修饰的。所以`blk`变量对block进行了强引用。

这时会调用`objc_retainBlock(blk)`来进行持有。而`objc_retaainBlock()`其实就是`Block_copy()`函数。

`Block_copy()`函数的作用是将栈上的block赋值到堆上，并将地址返回给blk变量。

所以变量引用一个栈区block，这个block会自动复制到堆区。

当这个方法执行完毕，blk变量被释放，block对象没有了强引用，对象也就被释放。

