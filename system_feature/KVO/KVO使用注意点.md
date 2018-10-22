
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