### load方法

#### 调用时机

系统启动之后读取这个类文件的时候就会调用这个类的`load`方法

#### 调用顺序

* 当本类和父类都实现了`load`方法时，父类的调用早于子类的调用

* 如果分类中也实现了`load`方法，当前类的`load`调用之后再去调用分类中的

#### 注意点

* 如果这个类没有实现`load`方法，就不会调用。不会继承父类的`load`方法。

* 不用写`[super load]`。如果父类实现了`load`方法，父类会在子类之前收到调用。

* 同一个库，类的加载顺序不确定。所以，不要在`load`方法中去使用其他的类。

* 有依赖关系的两个库，被依赖的库中的类会先被加载，`load`方法会先执行。

### initalize方法

#### 调用时机

这个方法是当这个类第一次被使用的时候被调用，runtime会向这个类发送`initalize`消息。

#### 调用顺序

* 如果子类没有实现`initalize`方法，会继承父类的方法实现。

* 顺序是先调用父类的，然后调用子类的

#### 注意点

子类执行`initalize`方法时，父类肯定已经执行过，所以不需要`super`

当子类继承父类的`initalize`方法时，父类的该方法可能会执行多次。如下

```
@interface BaseClass : NSObject
@end
@implementation BaseClass
+ (void)initialize {
    NSLog(@"%@ initalize", self);
}
@end

@interface SubClass : BaseClass
@end
@implementation SubClass
@end
```

输出 

```
>>> BaseClass initalize
>>> SubClass initalize
```

因为`SubClass`继承自`BaseClass`，所以在`SubClass`的使用时必定会先进行父类的使用。所以先调用父类的`initalize`方法

当子类使用时，子类继承了父类的方法实现，当时调用对象成为了当前这个子类。所以输出了`SubClass initalize`

### 两个方法的共同特点

* 开发者不主动调用的情况下，系统最多调用一次

* 如果子类和父类都被调用，父类的调用在子类之前

### 参考资料

* http://www.cocoachina.com/ios/20150104/10826.html

* http://www.cocoachina.com/ios/20150205/11113.html