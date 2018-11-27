# 单例模式

对于单例模式有两种方案。需要根据具体的业务场景选择所需要的方式。

1. 只提供单例方法。并不重写`alloc`等方法。

    这种情况类似于`NSNotificationCenter`的实现。当调用单例方法时会返回一个单例对象。但需要一个自定义的对象时也可以自己去创建。

2. 提供单例方法，并覆盖`alloc`等方法。保证无论何种方式拿到的都是单例对象。

单例模式的注意点是，需要保证在创建单例时要保证线程安全。不要发生当前线程未创建完对象其他线程也处理这个对象的情况

## 代码实现

```
static Singleton *_instance = nil;

+ (instancetype) shareInstance {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        _instance = [[self alloc] init];
    });
    return _instance;
}
```
