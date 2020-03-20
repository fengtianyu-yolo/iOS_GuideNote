# KVO 观察者机制

## KVO的用法

KVO是苹果提供的一套观察者机制。允许一个对象观察另一个对象的属性变化。

KVO和通知机制都是观察者模式的一种实现。KVO是一对一的方式，而通知是一对多的方式。

KVO可以监听单个属性的变化，也可以监听集合对象的变化。

**但是在监听集合对象的变化的时候需要通过`mutableArrayValueForKey:`方法获取集合对象**，这样集合对象内部发生变化时才会触发KVO

### 监听单个属性的变化

1. 注册观察者 `addObserver:forKeyPath:options:context:`
2. 观察者中实现`observeValueForKeyPath:ofObject:change:context:`方法，当观察到属性改变之后，会回调这个方法来通知观察者进行响应
3. 当观察者不需要监听的时候，移除观察者 `removeObserver:forKeyPath:`。这个方法需要在观察者对象销毁之前执行，否则会导致crash

#### 当前控制器监听`person.name`属性变化的示例

```

#import "ViewController.h"
#import "Person.h"

@interface ViewController ()
@property (nonatomic, strong) Person *person;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    self.person = [Person new];
    self.person.name = @"old_name";
    
    [self.person addObserver:self forKeyPath:@"name" options:NSKeyValueObservingOptionNew | NSKeyValueObservingOptionOld context:nil];
    
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context {
    NSLog(@"%@", keyPath);
    NSLog(@"%@", object);
    NSLog(@"%@", change);
}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    self.person.name = @"new_name";
}

@end
```

```
@interface Person : NSObject
@property (nonatomic, strong) NSString *name;
@end

```

### 监听数组变化的示例

```

#import "ViewController.h"
#import "Person.h"

@interface ViewController ()
@property (nonatomic, strong) Person *person;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    self.person = [Person new];
    self.person.mArray = [NSMutableArray array];
    [self.person.mArray addObject:@"first"];
    
    [self.person addObserver:self forKeyPath:@"mArray" options:NSKeyValueObservingOptionOld | NSKeyValueObservingOptionNew context:nil];
    
}

- (void)observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary<NSKeyValueChangeKey,id> *)change context:(void *)context {
    NSLog(@"%@", keyPath);
    NSLog(@"%@", object);
    NSLog(@"%@", change);
}

- (void)touchesBegan:(NSSet<UITouch *> *)touches withEvent:(UIEvent *)event {
    [[self.person mutableArrayValueForKey:@"mArray"] addObject:@"second"];
}

@end
```

```
@interface Person : NSObject
@property (nonatomic, strong) NSMutableArray *mArray;
@end
```

**通过`mutableArrayValueForKey`方法取得的数组对象去添加或移除元素时才会触发KVO机制。**

## KVO的实现原理

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

## 常见问题

### 如何关闭KVO机制

所谓关闭KVO也就是想对属性进行赋值但是并不通知观察者。

由KVO的实现原理可以知道，KVO的实现是通过混写`setter`方法，在子类的setter方法中调用`NSObject`的两个方法`willChangeValueForKey:`和`didChangeValueForKey:`来发出通知。

所以，想要避开KVO机制，其实就是赋值时不走setter方法即可。所以关闭KVO的方法也就是直接通过实例变量进行赋值，或KVC方式进行赋值。

### 手动触发KVO的方式

当添加观察者之后，通过点语法进行属性值的更改实质是调用的setter方法，所以使用上像是自动触发了KVO

想要手动触发KVO，模拟其底层实现。由于自动触发的时候是调用了`willChangeValueForKey:`和`didChangeValueForKey:`，所以手动触发的时候我们手动去调用这两个方法就可以触发观察者的回调方法。


### `removeObserver`和`addObserver`必须成对出现

`KVO`的`removeObserver`和`addObserver`应该是成对出现的。重复`removeObserver`会导致crash。而`addObserver`之后缺少`removeObserver`会导致已经释放的观察者去执行回调方法而crash

### 当父类子类都有观察者时

当父类子类都有观察者时，需要注意会先执行到子类的observe方法，此时如果子类的observe方法中在判断条件之后没有调用super，就会阻断观察者链，导致不会去执行父类的观察者方法

所以需要注意，在子类的keypath判断之后，当不是子类观察的属性之后，抛给父类去检查

* 父类

```
	// 父类
	- (void) viewDidLoad {
		Person *person = [Person new];
		[person addObserver:self forKeyPath:@"name" options:NSKeyValuesObservingOptionsNew context:nil];
	}

	- (void) observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *) context {
		NSLog(@"%@",change);
	}

	- (void) dealloc {
		[self.person removeObserver:self forKeyPath:@"name"];
	}
```

* 子类

```
	// 父类
	- (void) viewDidLoad {
	    Person *person = [Person new];
	    [person addObserver:self forKeyPath:@"age" options:NSKeyValuesObservingOptionsNew context:nil];
	    
	}

	- (void) observeValueForKeyPath:(NSString *)keyPath ofObject:(id)object change:(NSDictionary *)change context:(void *) context {
	    if ([keyPath isEqualToString:@"age"]) {
	        NSLog(@"%@",change);    
	    }else {
	        // 不是当前观察的属性 抛给父类去处理
	        [super observeValueForKeyPath:keyPath....]
	    }
	    
	}

	- (void) dealloc {
	    [self.person removeObserver:self forKeyPath:@"age"];
	}
```

### 父类和子类观察同一个属性

当父类和子类观察同一个属性时，在dealloc移除观察者时，会对该对象的同一个keyPath进行两次removeObserver操作，导致crash。

解决该问题可以通过context来进行标识，这样移除时可以移除自己的观察者，不会移除父类的观察者

```

- (void) viewDidLoad {
    Person *person = [Person new];
    [person addObserver:self forKeyPath:@"age" options:NSKeyValuesObservingOptionsNew context:@"ThisClassIsSon"];
    
}

- (void) dealloc {
    [self.person removeObserver:self forKeyPath:@"age" context:@"ThisClassIsSon"];   
}
```

