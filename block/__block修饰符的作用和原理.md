# __block修饰符的实现

## __block修饰的变量

当变量添加了`__block`修饰符之后，将代码

如下定义一个`__block`修饰的变量a
```
__block int a = 10

// <<<<<<<<<<<转换为C++ 代码>>>>>>>>>>>

__attribute__((__blocks__(byref))) __Block_byref_a_0 a = {(void*)0,(__Block_byref_a_0 *)&a, 0, sizeof(__Block_byref_a_0), 10};

// <<将代码中的类型转换去掉之后>>

__Block_byref_a_0 a = {(void*)0, &a, 0, sizeof(__Block_byref_a_0), 10};
```

通过c++代码可以看到，普通的变量`a`经过`__block`修饰之后，转换为c++之后定义成了一个`__Block_byref_a_0`结构体的实例。

`__Block_byref_a_0`结构体的定义如下

```
struct __Block_byref_a_0 {
  void *__isa;
__Block_byref_a_0 *__forwarding;
 int __flags;
 int __size;
 int a;
};
```

## block中为什么可以更改__block修饰的变量的值

当block中使用到`__block`修饰的变量时

```

    __block int a = 10;
    void (^blk)(void) = ^{
        a = 20;
    };
    
// <<<<<<<<<<<转换为C++ 代码>>>>>>>>>>> 

struct __main_block_impl_0 {
  
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;

  __Block_byref_a_0 *a; // by ref
    
  __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, __Block_byref_a_0 *_a, int flags=0) : a(_a->__forwarding) {
    impl.isa = &_NSConcreteStackBlock;
    impl.Flags = flags;
    impl.FuncPtr = fp;
    Desc = desc;
  }
    
};

static void __main_block_func_0(struct __main_block_impl_0 *__cself) {
  __Block_byref_a_0 *a = __cself->a; // bound by ref

        (a->__forwarding->a) = 20;
}

__attribute__((__blocks__(byref))) __Block_byref_a_0 a = {(void*)0,(__Block_byref_a_0 *)&a, 0, sizeof(__Block_byref_a_0), 10};

void (*blk)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, (__Block_byref_a_0 *)&a, 570425344));

((void (*)(__block_impl *))((__block_impl *)blk)->FuncPtr)((__block_impl *)blk);

```

从c++代码可以看到，当block中使用到了`__block`变量时，block对应的结构体中会增加属性。该属性为`__block`变量对应的结构体类型的指针。

当block初始化时，会将block中使用到的`__block`变量的结构体实例的地址赋值给block中的这个指针。

所以从代码中可以看到，当block中修改`__block`变量时，通过变量对应的结构体实例的指针去修改结构体中的成员。

## 为什么`__block`修饰的变量在变量作用域结束之后仍然可以使用。

在block中的静态变量中说到，静态变量是因为存储在了全局区，这块区域在系统运行时不会释放，所以可以通过保存变量的指针可以任何时间都能找到这个变量值，实现更改变量值。

而普通的变量因为当变量作用域结束之后变量会被释放，所以不能通过保存变量指针的方式更改变量值。

而`__block`的变量本质也是block中保存了这个变量对应结构体的指针。为什么这个变量在作用域结束之后不会被释放。

这和`__block`变量的存储区域有关。

默认情况下`__block`变量是配置在栈内存上。

若Block中使用到了这个`__block`变量，当Block从栈上复制到堆上的时候，这个`__block`变量也从栈上复制到了堆上。

所以，即使在该变量的作用域之外，`__block`变量由于被Block所持有，引用计数增加。所以也不会被释放。

只有当Block释放的时候，`__block`没有了持有者，这时候才会释放。

**当`__block`变量复制到堆上之后，为什么更改栈上的变量值，堆上的值也会改变**

如下代码
```
__block int val = 0;
void (^blk) (void) = [^{
  val += 10;
} copy];

val = 20;

blk();

NSLog(@"%d", val);
```

上面的代码执行后输出 30。

block中的`val += 10`使用val变量是复制到堆上的变量。

而`val = 20`仍然使用的是栈上的`val`变量。

但是当变量从栈上复制到堆上的时候，栈上的变量对应的结构体中的`forwarding`指针就指向了堆上的变量对应的结构体。*这个`forwarding`指针可以看做是`self`，就是指向自身的指针*

所以最终修改的都是堆上的结构体中的变量

## 总结

`__block`修饰的变量本质转为了一个结构体实例

block中可以更改该类型的变量是通过结构体的指针去修改结构体中的成员

`__block`变量可以在作用域之外使用是因为和block一起复制到了堆上。
