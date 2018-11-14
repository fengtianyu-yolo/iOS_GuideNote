
# 为什么全局变量、静态全局变量、静态变量的值可以在block更改

## 全局变量和静态全局变量

全局变量和静态全局变量是一样的，因为他们的作用域是全局的。

block实现转换为的函数中自然也可以访问到这些变量，所以也可以对其进行更改。

## 静态变量

静态变量和上面两种类型不一样。看静态变量转换后源码。

```

static int val = 2;
void (^blk)(void) = ^{
    val = 3;
    printf("fty log - val = %d", val);
};

blk();

// <<<<<<<<<<<转换为C++ 代码>>>>>>>>>>>

struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;

  int *val;

  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int *_val, int flags=0) : val(_val) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
};

static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
  int *val = __cself->val; // bound by copy

        (*val) = 3;
        printf("fty log - val = %d", (*val));
}

int main(int argc, const char * argv[]) {

    static int val = 2;
    void (*blk)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, &val));

    ((void (*)(__block_impl *))((__block_impl *)blk)->FuncPtr)((__block_impl *)blk);

    return 0;
}
```

可以看到，当block使用到静态变量时，该block对应的结构体中增加了一个属性，这个属性是使用到的静态变量类型的指针。

在结构体的构造函数也可以看到，初始化这个结构体时，也是将静态变量的地址传递给结构体。

所以。当block中更改静态变量时，是通过静态变量的指针去更改静态变量的值。

