
### `strong`和`copy`修饰`NSString`属性时的不同

iOS中对字符串的修饰可以用`strong`也可以用`copy`,如下所示

```
@property (nonatomic, copy) NSString *strCopy;
@property (nonatomic, strong) NSString *strStrong;
```

这个两种修饰符的区别如下

* 当对这个属性赋值一个不可变的`NSString`对象时，作用相同，都是对字符串对象地址的浅拷贝。由于这个字符串是个不可变类型，所以，属性值也不会变。

* 当对这个属性赋值一个可变的`NSMutableString`对象时，`copy`修饰的属性是深拷贝，`strong`修饰的属性是浅拷贝。当原字符串更改，copy的属性值不会改变，而strong修饰的属性的属性值会随着改变。
    * 注意`appendString`才是对原字符串的改变，`stringByAppendingString`等则是返回一个新的字符串对象。

所以综合上面的情况，更推荐使用`copy`来修饰字符串属性，提高属性的安全性。为这个属性赋值一个`NSString`对象两种效果是相同的，所以可以用copy，而一旦是赋值一个`NSMutableString`时候，`copy`的属性可以保证属性值不会随赋值时的字符串对象的变化而变化，更优于`strong`修饰。

### 证明

#### 属性被赋值不可变字符串

`NSString = NSString`

```
@interface ViewController ()
@property (nonatomic, strong) NSString *strStrong;
@property (nonatomic, copy) NSString *strCopy;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    NSString *text = @"a string object";
    self.strStrong = text;
    self.strCopy = text;
    
    NSLog(@"原字符串地址       == %p", text);
    NSLog(@"strong属性地址   == %p", self.strStrong);
    NSLog(@"copy属性地址      == %p", self.strCopy);
}
```

##### 输出

```
>>> 原字符串地址   ==  0x10c626078
>>> strong属性地址  == 0x10c626078
>>> copy属性地址 ==    0x10c626078
```

所以证明，当`text`是不可变的`NSString`类型时，这种情况下都是同一个指针地址，指向的是同一个对象。两种修饰符的效果是一样的。都是浅拷贝。

##### 注意

需要注意的是下面这种情况

```
- (void)viewDidLoad {
    [super viewDidLoad];
    
    NSString *text = @"a string object";
    self.strStrong = text;
    self.strCopy = text;
    
    text = @"a new string object";
    
    NSLog(@"%@", text);
    NSLog(@"%@", self.strStrong);
    NSLog(@"%@", self.strCopy);
}

```
这种情况下的输出是 

```
>>> a new string object 
>>> a string object
>>> a string object
```

因为当`text = @"a new string object"`的时候，text指针指向了新的字符串对象。而`strStrong`和`strCopy`属性的指针仍然是指向了原来的`a string object`字符串对象。他们的指针是不会跟着变化的


***

#### 属性被赋值可变字符串对象时

`NSString = NSMutableString`

```
- (void) mutableStringTest {
    NSMutableString *text = [[NSMutableString alloc] initWithString:@"a mutable string object"];
    self.strStrong = text;
    self.strCopy = text;
    
    NSLog(@"原字符串地址       == %p", text);
    NSLog(@"strong属性地址   == %p", self.strStrong);
    NSLog(@"copy属性地址      == %p", self.strCopy);
    
    [text appendString:@" after modify"];

    NSLog(@"%@", text);
    NSLog(@"%@", self.strStrong);
    NSLog(@"%@", self.strCopy);
}

```

##### 输出

``` 
>>> 原字符串地址   ==  0x60400025c0b0
>>> strong属性地址  == 0x60400025c0b0
>>> copy属性地址 ==    0x60400025c3b0

>>> a mutable string object after modify
>>> a mutable string object after modify
>>> a mutable string object
```

可以看到，当属性被赋值可变字符串的时候，`copy`属性会拷贝出来一个新的对象，而`strong`属性则是对指针地址的拷贝。所以，当原字符串发送了改变的时候，`copy`属性的值并不会发送变化。而`strong`属性的值发送了变化。

在这种情况下两种修饰符的效果是不同的。

***

### 为什么`copy`修饰的字符串不同类型赋值时会产生深浅拷贝

因为当`copy`修饰的属性，对它进行赋值的时候，它的`setter`方法中执行的是`copy`操作。如下所示

```
- (void)setName:(NSString *)name {
    _name = [name copy];
}
```

因为是执行的`copy`操作，所以产生了`NSString`和`NSMutableString`执行`copy`方法产生的不同结果。

当`NSString`执行`copy`方法，返回的是仍然是`NSString`类型，是做了一次浅拷贝，只拷贝了字符串对象的地址

而当`NSMutableString`执行`copy`方法，返回的则是`NSString`类型，这做的是一次深拷贝。

所以，无论是可变类型还是不可变类型字符串，进行`copy`操作都是复制出来一个不可变类型的字符串，区别在于是否拷贝出来新的对象。

### `copy`方法和`mutableCopy`方法

既然有`copy`方法，同样还有`mutableCopy`方法，这两个方法的特点如下

* 拷贝之后的对象类型的区别
    
    * `copy`方法拷贝出来的都是不可变类型 
        
        > `[NSMutableArray copy] -> NSArray`
        
        > `[NSMutableString copy] -> NSString`
        
        > `[NSString copy] -> NSString`

    * `mutableCopy`方法拷贝出来的都是可变类型
        
        > `[NSString mutableCopy] -> NSMutableString`
        
        > `[NSArray mutableCopy] -> NSMutableArray`
        
        > `[NSMutableString copy] -> NSMutableString`

* 是否产生新对象的区别

    * `mutableCopy`做的都是深拷贝，得到的都是一个新的对象

    * `copy`方法只有对可变类型进行操作时是深拷贝，对不可变类型进行copy是浅拷贝
    
### `copy`可以描述什么样的属性

通过上面可以知道，`copy`修饰符修饰的属性在`setter`方法中执行的`[xx copy]`操作

所以，如果要调一个对象的`copy`方法，那么这个对象必须要遵守`NSCopying`协议。这个协议中规定了一个方法`-(id)copyWithZone:(NSZone *)zone`

通过实现这个方法给对象提供拷贝的功能。在这个方法决定copy时是进行深拷贝还是浅拷贝。

`NSString` 、 `NSDictionary`等默认已经实现了这个方法。

*** 

### 如果属性是`NSMutable`，一定不要用`copy`修饰！！

下面是一段示例代码

```
@interface ViewController ()
@property (nonatomic, copy) NSMutableString *myStr;
@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    
    NSMutableString *mStr = [NSMutableString stringWithFormat:@"aa"];
    self.myStr = mStr;
    
    if ([self.myStr isKindOfClass:[NSMutableString class]]) {
        NSLog(@"1");
    }else if([self.myStr isKindOfClass:[NSString class]]){
        NSLog(@"2");
    }

    [self.myStr appendString:@"aa"];
}

```

上面代码的运行结果是，先输出2，然后crash。报出来`-[NSTaggedPointerString appendString:]: unrecognized selector sent to instance`

#### 原因分析

从上面的原理可以看到，如果`copy`修饰，`setter`中会执行`copy`操作。而可变类型的`copy`操作返回的是不可变类型。

所以`self.myStr`被赋值之后变成了不可变类型的`NSString`。

所以先输出了2

`appendString`方法是`NSMutableString`类中的，`NSString`调用它所以报出来未找到方法的异常。
