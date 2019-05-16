# 通过KVC可以对数组进行的操作

## 数组中的数据**去重**

`@distinctUnionOfObjects`

```
    NSArray *array = @[@"key", @"value", @"result", @"key"];
    NSArray *result = [array valueForKeyPath:@"@distinctUnionOfObjects.self"];
    NSLog(@"%@", result);

    >>> (result,value,key)
```

**不要忽略添加`@`符号**

### 当数组中的元素是自定义对象时，根据对象的指定属性来去重

```
    Person *p1 = [[Person alloc] initWithName:@"apple" Age:12];
    Person *p2 = [[Person alloc] initWithName:@"dell" Age:8];
    Person *p3 = [[Person alloc] initWithName:@"lg" Age:19];
    Person *p4 = [[Person alloc] initWithName:@"apple" Age:19];

    NSArray *result = [personArray valueForKeyPath:@"@distinctUnionOfObjects.name"];
    NSLog(@"%@", result);

    >>> (lg,dell,apple)
```

## 对数组中的元素**求和**

`@sum`


