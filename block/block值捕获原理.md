# block中的捕获外部变量值的原理

## 为什么捕获的是变量的值

由block的本质可以知道，block转换之后是一个结构体。

当没有使用到外部变量时的block转换后的结构体类型如下所示，结构体中只有两个属性`impl`和`desc`

```
void (^blk)(void) = ^{
    printf("fty log - blk");
};

//<<<<<<<<<<<< 转换为c++ >>>>>>>>>>>>>>>>>

struct __main_block_impl_0 {
  
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
    
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int flags=0) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};
```

当block中使用到block外的变量时，在结构体中就会增加与外部变量相同类型的一个变量。如图所示

```

int val = 2;
void (^blk)(void) = ^{
    printf("fty log - val = %d", val);
};

//<<<<<<<<<<<< 转换为c++ >>>>>>>>>>>>>>>>>

struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;

  int val;

  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int _val, int flags=0) : val(_val) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};

static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
  int val = __cself->val; // bound by copy
        printf("fty log - val = %d", val);
}
```

定义一个block本质是通过结构体的构造函数创建一个对应结构体的实例。

而当block中使用到外部变量，外部变量作为构造函数的参数传递进来，赋值给结构体中对应的变量进行保存。

所以，当构造函数执行完毕之后，外部变量的值进行更改，并不会影响结构体中的变量的值。

## 为什么不能更改外部变量的值

由上面可以知道，block中获取的外部变量的值是在初始化结构体时复制了一份该变量的值自己保存。并不是变量的指针。

在block对应的实现函数中可以看到，变量的读取操作也是读取的结构体中的这个变量。所以在block中是无法更改外部变量的。

编译器为了避免这种操作，所以在当尝试在block中更改使用到的外部变量时会编译报错。


