
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

所以综合上面的情况，更推荐使用`copy`来修饰字符串属性，提高属性的安全性。为这个属性赋值一个`NSString`对象两种效果是相同的，而一旦是赋值一个`NSMutableString`时候，`copy`的属性可以保证属性值不会随赋值时的字符串对象的变化而变化。

### 证明

#### 属性被赋值不可变字符串

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
所以，这种情况下都是同一个指针地址，指向的是同一个对象。

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

可以看到当属性被赋值可变字符串的时候，copy属性会拷贝出来一个新的对象，而strong属性则是对指针地址的拷贝。所以，当原字符串发送了改变的时候，copy属性的值并不会发送变化。而strong属性的值发送了变化。

***
***

对于属性为`NSMutableString`时，情况是一样的。
