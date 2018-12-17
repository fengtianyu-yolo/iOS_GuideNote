
在OC中用于指明一个字段的值是否可以为空可以通过下面关键字来标识。

`_Nullable` `_Nonnull`

`__nullable` `__nonnull`

`nullable` `nonnull`

这三种关键字本质是一样的。

在XCode6.3引入的`__nullable`和`__nonnull`关键字用于指明对象是否可以为空。

为了避免和第三方库中有冲突。在XCode7中引入了`_Nullable`和`_Nonnull`。从而建议弃用`__nullable`和`__nonnull`

并且苹果同样支持`nonnull`和`nullable`的写法。

所以有了三种方式的关键字来指明是否可以为空。不过三种关键字的放置位置不同。

## nonnull 和 nullable的使用

`nonnull`和`nullable`使用时一般放在类型前面

* 修饰属性时 

```
@property (nonatomic, strong, nonnull) NSString *name;
```

* 修饰参数时

```
-(void)methodWithString:(nullable NSString *)string;
```

* 修饰方法返回值时

```
- (nullable NSString *)method;
```

**需要注意的是这个关键字不能用于修饰block的参数、返回值等**

## _Nonnull 和 _Nullable的使用

`_Nonnull`和`_Nullable`使用时放在类型后面

* 修饰属性时

```
@property (nonatomic,strong) NSString * _Nullable string;
```

* 修饰参数时

```
- (void)methodWithString:(NSString *_Nullable)string;
```

* 修饰方法返回值时

```
- (NSString * Nullable)method;
```

* 修饰Block的参数

```
- (void)methodWithBlock:(void (^)(id _Nonnull))block;
```

## 总结使用方式

1. 方法返回值、方法参数、属性的声明时使用`nonnull`和`nullable`

2. block中的参数、block返回值的修饰用`_Nullable`和`_Nonnull`

# 简化声明空值类型的宏

升级XCode10之后创建文件时会自动添加这对宏`NS_ASSUME_NONNULL_BEGIN` `NS_ASSUME_NONNULL_END` 如下所示

```
NS_ASSUME_NONNULL_BEGIN

@interface MyObject : NSObject
@end

NS_ASSUME_NONNULL_END
```
**这对宏定义的意思是在这对宏之间的所有属性、参数、返回值等都是不可空的。如果需要可以为空需要单独指明**

