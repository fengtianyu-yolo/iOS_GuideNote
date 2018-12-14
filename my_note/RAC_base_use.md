# RAC常用API使用

## 常用宏

### RAC(object, keypath)

#### 用法一

RAC(对象, 属性) = 信号

作用是当信号发出next时，将信号中的值赋值到对象的属性上

#### 用法二

RAC(对象, 属性, 当空时使用该值) = 信号

当这个信号中的数据是`nil`时，使用第三个参数中的值

#### 例子

当`textField`输入框中文本发生变化时更改`content`属性的值

```
RAC(self, content) = self.textField.rac_textSignal;
```

### RACObserve()

它也是一个信号，监听对象中指定属性的变化，订阅这个信号，当属性值发生变化时，就可以获取属性的值

#### 用法

[RACObserve(对象, 属性) subscribeNext:(id x){}]

#### 例子

```
[RACObserve(self, contnet) subscribeNext:(id x){
    NSLog(@"new value %@", x);
}]
```

## 常用API

### RACCommand 

## UIKit 扩展

### UIButton

#### rac_command

当按钮点击时自动执行这个command，按钮的是否可以点击取决于这个命令的`canExecute`

##### 例子

```
[UIButton new].rac_command = [[RACCommand alloc] initWithSignalBlock:^RACSignal *(id input) {
    // do somethings
    return [RACSignal empty];
}];
```
当需要把数据传递出去的时候，在最后创建信号的时候把数据send出去即可。

#### rac_signalForControlEvents:

当点击按钮的时候发出信号

##### 例子

```
[[self.btn rac_signalForControlEvents:UIControlEventTouchUpInside] subscribeNext:(id x){
    // do somethings
}];
```

## 对信号的处理

filter map reduce