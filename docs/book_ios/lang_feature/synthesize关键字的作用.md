通常我们习惯于如下方式定义一个属性，并通过点语法来调用属性的`getter`和`setter`方法。有时当重写`getter`__或__`setter`方法时还会使用带下划线的同名实例变量`_str`。那么，`getter`和`setter`以及这个实例变量是哪里来的。

```objective-c
@property (nonatomic, strong) NSString *str
```

iOS6之后 llvm编译器引入`property autosysthesis` 属性自动合成。    

也就是说`property`定义的属性会自动添加这样一行代码 `@synthesize propertyName = _propertyName`    

这也就是为什么`property`属性可以用带下划线的属性


`@synthesize propertyName = _propertyName`   
这行代码的作用是
1. 给`propertyName`这个属性生成`getter`和`setter`方法。
2. 同时生成一个带下划线前缀的实例变量。或者说给`propertyName`添加一个别名

***

### 什么时候会用到`synthesize`

当下面这些情况的时候，系统不会自动的进行属性合成。也就是不会自动的生成`getter`和`setter`方法也不会自动的生成带`_`的实例变量。而我们需要使用实例变量和相关方法的时候，就需要手动添加`synthesize`来合成实例变量。

1. 同时重写getter和setter方法时，不会自动合成属性。__当我们只重写其中一个的时候还是会进行属性自动合成的__
2. 重写了只读属性的getter方法时。即表示当重写了`readonly`属性的`getter`方法时，带`_`的实例变量需要我们手动通过`synthesize`来合成。
3. 使用了`@dynamic`时
4. 协议中的属性
5. _category中的属性_
6. _重载的属性_

下面来分别说明

#### 协议中的属性
协议中定义了属性，协议中定义了属性其实也就是定义了该属性的getter和setter方法，但是并没有该方法的实现。

所以在遵守该协议的类中应该实现对应的getter和setter方法。可以直接通过`synthesize`关键字来自动生成。


#### `@dynamic`

用`property`会自动的添加`synthesize`,而`synthesize`的作用是自动生成getter和setter，并且定义一个带下划线的实例变量

而 `dynamic`关键字则会阻止自动生成getter和setter方法和生成变量

```objective-c
@interface model
@property (nonatomic, strong) NSString *str;
@end

@implementation
@dynamic str;
@end
```
