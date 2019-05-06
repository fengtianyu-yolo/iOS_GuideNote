
# RACCommand 

`RACComand`可以用来进行一些操作，在操作完成之后将数据发送出去,比如按钮的点击响应

## 使用场景：点击按钮的事件响应

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

## 怎么调用`RACComand`,来让这个操作去执行

上面是创建了一个`RACCommand`。这个命令的执行可以通过调用`execute`方法。如下所示。

当`viewController`中的按钮点击之后，执行这个command

```
UIButton *btn = [UIButton new] rac_signalForControlEvents:UIControlEventTouchUpInside] subscribeNext:^(UIButton *btn) {
            [vm.command execute];
        }];
```

### 怎么给command传递参数

```
[vm.command execute:];
```

对于`UIButton`还可以直接通过RAC中`UIButton`的分类提供的`rac_command`,将`command`赋值给该属性，如下所示。

```
UIButton *btn = [UIButton new];
btn.rac_command = vm.command;
```

这样当点击按钮之后就会去执行这个command。

## 获取command执行的结果

### 第一种获取command执行结果数据的方式

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

### 第二种获取command执行结果数据的方式 - `executionSignals`属性 

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

## 其他的属性

`executing`

`enable`