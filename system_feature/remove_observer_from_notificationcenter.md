# 通知中心的监听者是否需要主动移除

## 默认情况下的通知使用方式

在涉及使用通知中心时，通常添加监听和移除监听的操作。如下所示

添加
```
[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(action) name:@"" object:nil];
```

移除监听

```
- (void) dealloc {
   [[NSNotificationCenter defaultCenter] removeObserver:self]; 
}
```

这是因为当通知中心`addObserver`时，通知中心会持有当前监听者。

当通知中心接收到消息时，会检查自己持有的监听者中是否有需要响应该消息的。

如果有，则调用该监听者的对应方法。

**在iOS9之前，通知中心是通过`unsafe_unretained`指针引用的监听者。那么当该监听者释放之后，这个指针就指向了一个空地址，成为一个野指针。**

**所以，需要监听者在自己释放时，主要将自己从通知中心移除。**

## iOS9 之后

iOS9之后，通知中心通过`weak`指针来引用监听者。这样，当监听者释放之后，该指针会自动置为`nil`。

所以，不再需要主动移除。

## 注意这个方法

当使用`block`回调方式的方法时，需要手动移除监听者。

如其注释，这个方法会使监听者被系统持有。

```
// The return value is retained by the system, and should be held onto by the caller in
// order to remove the observer with removeObserver: later, to stop observation.

- (id <NSObject>)addObserverForName:(nullable NSNotificationName)name object:(nullable id)obj queue:(nullable NSOperationQueue *)queue usingBlock:(void (^)(NSNotification *note))block API_AVAILABLE(macos(10.6), ios(4.0), watchos(2.0), tvos(9.0));
```
