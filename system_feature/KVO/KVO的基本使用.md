
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

通过`mutableArrayValueForKey`方法取得的数组对象去添加或移除元素时才会触发KVO机制。