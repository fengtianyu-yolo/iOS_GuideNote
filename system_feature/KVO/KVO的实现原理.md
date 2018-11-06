
### KVO的基本实现原理是这样的

1. 在添加观察者`addObserver`的时候，该方法内部其实是通过runtime创建一个被观察对象类的子类
2. 将原对象的isa指针指向新创建的子类。将isa指针指向新的子类，调用方法也就调用了新的子类的方法。同时重写这个子类的class方法，使其返回父类。*这一步是最关键的一步,让调用者在无感知的情况下调用了子类的方法*
3. 重写子类的setter方法
4. 子类执行setter方法的时候，进行回调方法


### 手动实现KVO

按照上面的步骤实现一个KVO

KVO过程中首先要添加观察者，像下面这样。基于KVO的实现原理，我们模仿该方法的实现。

```
    [self.person addObserver:self forKeyPath:@"name" options:NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld context:nil];
```


在这一步中，可以看到`addObserver`这个方法是在`NSObject+NSKeyValueObserverRegistration`分类中。

那同样也创建一个类似这样的分类 `NSObject_MyKeyValueObserver`。

在这个分类提供一个添加观察者方法`ty_addObserver: forKeyPath: options: context:`

在这个方法中，我们动态创建一个被观察者对象类的子类，然后将被观察者对象的isa指针指向子类。然后重写class方法。

```

#import "NSObject+MyObserver.h"
#import <objc/runtime.h>
#import <objc/message.h>
@implementation NSObject (MyObserver)

- (void) ty_addObserver:(id)observer  forKeyPath:(NSString *)key {
    
    // 1 获取当前类
    
    // 获取当前被观察者对象的类
    Class class = object_getClass(self);
    // 获取当前被观察者对象的类的类名
    NSString *className = NSStringFromClass(class);
    
    // 2 创建当前类的子类
    
    // 判断当前类类名是否已经是KVO_开头的
    if (![className hasPrefix:@"KVO_"]) {
        // 创建当前被观察对象类的子类
        NSString *kvoClazzName = [@"KVO_" stringByAppendingString:className]; // 子类类名为KVO_+原类名
        Class cls = NSClassFromString(kvoClazzName); // 获取子类
        
        if (cls) {
            // 如果这个类已经存在 ， 将当前被观察者对象的isa指针指向这个子类
            object_setClass(self, cls);
        } else {
            // 获取当前被观察者对象的类
            Class initialClass = object_getClass(self);
            // 创建当前类的子类
            Class kvoClass = objc_allocateClassPair(initialClass, kvoClazzName.UTF8String, 0);
            // 创建类的class方法
            Method classMethod = class_getInstanceMethod(kvoClass, @selector(class));
            // 获取方法的Type字符串 (包含参数类型 和 返回值类型)
            const char *types = method_getTypeEncoding(classMethod);
            // 为子类添加class方法
            class_addMethod(kvoClass, @selector(class), (IMP)classMethod, types);
            // 注册这个新的子类
            objc_registerClassPair(kvoClass);
        }
    }
    
    // 3 重写setter方法
    SEL setterSelector = NSSelectorFromString(key);
    Method setterMethod = class_getInstanceMethod([self class], setterSelector);
    const char *types = method_getTypeEncoding(setterMethod);
    class_addMethod(class, setterSelector, (IMP)kvo_setter, types);
}

static void kvo_setter(id self, SEL _cmd, id newValue) {
    
    NSString * setterName = NSStringFromSelector(_cmd);
    NSString * getterName = getterForSetter(setterName);
    
    struct objc_super superclass = {
        .receiver = self,
        .super_class = class_getSuperclass(object_getClass(self))
    };
    
    
    // 发送通知
    [self willChangeValueForKey:getterName];
    // 向父类发送set消息
    void (*objc_msgSendSuperCasted)(void *, SEL, id) = (void *)objc_msgSendSuper;
    objc_msgSendSuperCasted(&superclass, _cmd, newValue);
    // 调用完父类的setter方法后，发送通知
    [self didChangeValueForKey: getterName];
}

@end


```



