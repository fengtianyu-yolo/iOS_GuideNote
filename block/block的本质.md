# Block的本质


## Block转换为C++

这样一段代码

```
int main(int argc, const char * argv[]) {
    void (^blk)(void) = ^{
        printf("Block");
    };
    blk();
    return 0;
}
```

经过clang转换为c++代码之后

```
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

struct __block_impl {
  void *isa;
  int Flags;
  int Reserved;
  void *FuncPtr;
};

static struct __main_block_desc_0 {
  size_t reserved;
  size_t Block_size;
} __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};

static void __main_block_func_0(struct __main_block_impl_0 *__cself) {

        printf("Block");
}

int main(int argc, const char * argv[]) {
    void (*blk)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA));
    
    ((void (*)(__block_impl *)) ((__block_impl *)blk)->FuncPtr) ((__block_impl *)blk);
    return 0;
}
```

将代码拆开来看

OC中定义一个Block
```
    void (^blk)(void) = ^{
        printf("Block");
    };
```

转换成了下面的代码。

```
    void (*blk)(void) = ((void (*)())&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA));
```    
去掉类型转换，将代码简化

```
    void(*blk)(void) = __main_block_impl_0(__main_block_func_0, &__main_block_desc_0_DATA)
```

从上面可以看到，`=`后面是创建一个Block的部分，转换之后创建Block的过程其实就是调用了`__main_block_impl_0`结构体的构造方法生成一个结构体的实例。

这个构造方法中传入了两个参数。第一个是`__main_block_func_0`，从上面的代码可以看到，这是一个函数指针。这个函数就是Block的具体实现转换成的函数。第二个参数是一个结构体实例的指针。这个参数是确定了结构体所需要的空间大小。

通过上面分析可以看到，定义一个block本质是定义了一个结构体实例。

接下来看`=`左边的部分。

`=`左边定义了一个Block类型的`blk`变量。从上面可以看到右边是调用了结构体的构造函数，那么自然生成的是结构体的实例，所以Block类型的变量其实也就是一个结构体的指针。`void(^blk)(void)` == `struct __main_block_impl_0 *blk`

**block对象转换为c++之后其实就是一个结构体实例，block变量就是一个结构体实例的指针**

### 解析`__main_block_impl_0`结构体

既然Block就是一个`__main_block_impl_0`的实例，那么看看这个结构体的具体内容

```
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
这个结构体有两个属性和一个构造方法，去掉构造方法之后，这个结构体就只有两个属性。

```
struct __main_block_impl_0 {
  struct __block_impl impl;
  struct __main_block_desc_0* Desc;
};
```
第一个属性是一个结构体，这个结构体的声明如下
```
struct __block_impl {
  void *isa;
  int Flags;
  int Reserved;
  void *FuncPtr;
};
```
这个结构体定义了一个函数指针的属性，这个指针指向了block实现转换后的函数。

还有一个isa指针。用于指向该block所属的类型

### block的转换后代码对比和关系如图所示

![block essence](https://github.com/cocacola-ty/Images/blob/master/block_essence.png?raw=true)