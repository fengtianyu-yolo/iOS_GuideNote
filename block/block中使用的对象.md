# Block中使用到的对象

```
blk_t blk;
{
    id array = [[NSMutableArray alloc] init];
    blk = [^(id obj){
        [array addObject:obj];
        NSLog(@"%d", array.count)
    } copy];
}

blk([NSObject new]);
blk([NSObject new]);
blk([NSObject new]);
```

上面的代码最终结果会输出 1 2 3

## 为什么`array`在它的作用域结束之后，block中仍然可以使用这个对象

当block中使用到了对象

block对应的结构体中就会增加一个属性，用来保存这个对象的指针。如下所示。

```
struct __main_block_impl_0 {
    struct __block_impl impl;
    struct __main_block_desc_0* Desc;

    id __strong array;
}
```
同时会增加下面几个方法
```
static void __main_block_copy_0 (struct __main_block_impl_0 *dst,struct __main_block_impl_0 *src ) {
    _Block_object_assign(&dst->array, src->array, BLOCK_FIELD_IS_OBJECT);
}

static void __main_block_dispose_0 (struct __main_block_impl_0 *dst,struct __main_block_impl_0 *src ) {
    _Block_object_dispose(&dst->array, src->array, BLOCK_FIELD_IS_OBJECT);
}
```

当block中使用到了对象，在block创建的时候，会将对象的地址赋值给结构体中的对应指针。而此时block对应的这个结构体并没有持有这个对象。

而是通过调用`__main_block_copy_0`这个函数对block中的对象进行持有。

这个函数的调用时机就是在block从栈上复制到堆上的时候。

上面的示例代码中，如果block没有调用copy方法进行复制到堆上的操作。那么程序就会crash。因为array对象已经释放了。