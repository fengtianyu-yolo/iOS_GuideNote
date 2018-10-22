
### 如何关闭KVO机制

所谓关闭KVO也就是想对属性进行赋值但是并不通知观察者。

由KVO的实现原理可以知道，KVO的实现是通过混写`setter`方法，在子类的setter方法中调用`NSObject`的两个方法`willChangeValueForKey:`和`didChangeValueForKey:`来发出通知。

所以，想要避开KVO机制，其实就是赋值时不走setter方法即可。所以关闭KVO的方法也就是直接通过实例变量进行赋值，或KVC方式进行赋值。

### 手动触发KVO的方式

当添加观察者之后，通过点语法进行属性值的更改实质是调用的setter方法，所以使用上像是自动触发了KVO

想要手动触发KVO，模拟其底层实现。由于自动触发的时候是调用了`willChangeValueForKey:`和`didChangeValueForKey:`，所以手动触发的时候我们手动去调用这两个方法就可以触发观察者的回调方法。