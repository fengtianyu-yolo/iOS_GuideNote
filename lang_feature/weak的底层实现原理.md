
### 简述

runtime维护了一个hash表，表的key是对象的地址，value是一个数组，数组中是这个对象的weak指针。

当一个weak指针指向对象的时候，会根据对象地址去这个weak表中查找是否已经存在

如果存在将这个新的weak指针添加到weak表中以该对象地址为key对应的数组中。

如果不存在则在weak表中以该对象地址为key新加一条记录。

### 详细过程

当一个weak指针指向一个对象的时候，runtime首先会调用`objc_initWeak`函数，这个函数会简单判断这个对象是否有效，无效直接将指针置为`nil`然后返回。有效继续向下调用

接下来会调用`objc_storeWeak()`函数，这个函数中则进行了

### 当`weak`指针指向的对象释放时，`weak`指针如何自动置为`nil`的

当weak指针指向的对象进行`release`操作时，调用的是`objc_release`,这时引用计数减1

如果引用计数减1之后成为0，则该对象没有引用，所以执行`dealloc`操作

函数向下执行会调用`_objc_rootDealloc` -> `object_dispose` -> `objc_destructInstance` -> `objc_clear_deallocating`

在这个`objc_clear_deallocting`函数中，会进行如下的操作将指针置为`nil`

    1. 在weak表中获取释放对象的地址为key的这个记录
    2. 将这个key对应value中的所有weak指针的地址置为nil
    3. 在weak表中删除这个记录
    4. 引用计数表中删除以该对象地址作为key的记录

