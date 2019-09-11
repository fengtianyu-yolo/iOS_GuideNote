
# NSOperation

## 参考链接

* https://www.jianshu.com/p/4b1d77054b35

## `NSOperation`的基本使用方式

```

- (void)operation {
	NSOperation *operation = [NSOperation ];
	NSOperationQueue *queue = [[NSOperationQueue alloc] init];

}

```

## 选择使用`NSOperation`的原因

* 可以设置任务完成之后的操作

* 可以设置任务之间的依赖关系

* 可以设置任务的优先级

* 可以取消任务的执行

* 可以使用KVO来观察任务的执行状态，`isExecuteing`、`isFinished`、`isCancelled`

### 任务之间依赖的场景：接口之间的顺序执行

### 观察任务状态的场景：
