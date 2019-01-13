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

#### 使用场景：点击按钮的事件响应

比如，当点击登录按钮之后去进行登录请求的过程。

在ViewModel中创建一个`RACCommand`。这个`RACCommand`中进行登录请求的操作。代码如下所示。

```
RACCommand *command =  [[RACCommand alloc] initWithSignalBlock:^RACSignal *(id input) {
        
        return [RACSignal createSignal:^RACDisposable *(id<RACSubscriber> subscriber) {
            NSMutableDictionary *param = [NSMutableDictionary dictionary];
            [param setValue:self.code forKey:@"code"];
            [[HTTPFacade sharedInstance] doHttpRequestWithParams:param uri:support_random_codeuse completeBlock:^(NSError *error, id resultObject) {
                if (error) {
                    [subscriber sendNext:error];
                    [subscriber sendCompleted];
                } else {
                    [subscriber sendNext:@YES];
                    [subscriber sendCompleted];
                }
            }];
            return nil;
        }];
        
    }];
```

`RACCommand`的返回值是一个信号。所以可以将请求的结果值由最后的信号发出去。**这里需要注意：数据发送完毕之后必须调用`[subscriber sendCompleted]`否则这个命令一直处于执行中。**

上面是创建了一个`RACCommand`。这个命令的执行是通过下面的方式。

当`viewController`中的按钮点击之后，执行这个command

```
UIButton *btn = [UIButton new] rac_signalForControlEvents:UIControlEventTouchUpInside] subscribeNext:^(UIButton *btn) {
            [vm.command execute];
        }];
```

或者通过RAC中`UIButton`的分类提供的`rac_command`

```
UIButton *btn = [UIButton new];
btn.rac_command = vm.command;
```

这样当点击按钮之后就会去执行这个command。

##### 第二种数据传递方式

上面的方式是将请求结果放在`RACCommand`返回的信号中。也可以在viewModel中定义一个属性来存放返回的结果值。这样vc中就不用去订阅`RACComand`的信号了，监听存放数据的属性值的变化就行了。如下所示。

`ViewModel`

```
RACCommand *command =  [[RACCommand alloc] initWithSignalBlock:^RACSignal *(id input) {
        
     [[HTTPFacade sharedInstance] doHttpRequestWithParams:param uri:support_random_codeuse completeBlock:^(NSError *error, id resultObject) {
            if (error) {
           		self.requestSuccess = NO;
            } else {
            	self.requestSuccess = YES;
            }
        }];
        return [RACSignal empty];        
    }];
```

`ViewController`

```
[RACObserve(viewModel, requestSuccess) subscribeNext:(id x) {

}];
```

#### `executionSignals`属性 

这个信号就是command执行之后返回的信号。

比如在上面的例子中就可以通过这个属性来获取`RACComand`执行的值。如下所示。

`ViewController`

```
UIButton *btn = [UIButton new];
[btn addTarget:self action:@selector(action) forControlEvents:UIControlEventTouchUpInside];

- (void)action {

	[[self.viewModel.command.executionSignals switchToLatest] subscribeNext:^(id response) {

	}];

	[self.viewModel.command execute:nil];
}
```

这里的使用有两个注意点：

1. `executionSignals`是一个冷信号，所以需要先订阅再使用。**如果先`command execute:`，就接收不到这个信号。**

2. `executionSignals`是一个信号的信号。也就是说，这个信号的内容是一个信号。真正的数据内容在最里面的这个信号中。

	所以，这里`switchToLatest`方法的作用是获取信号中的最新的信号。

	如果不使用`switchToLatest`也可以如下方式获取信号中的内容。

	```
	- (void)action {
		[self.viewModel.command.executionSignals subscribeNext:^(id response) {
			// 订阅信号中的内容
			[response subscribeNext:^(id data) {
				NSLog(@"%@",data);
			}];
		}];
	}
	```

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