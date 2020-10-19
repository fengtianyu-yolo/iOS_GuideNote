# 动画的实现实践

常见的动画实现方式: `CoreAnimation`、UIView动画块、UIView动画事务、隐式动画

## CAAnimation

- 使用`CAAnimation`必须要赋值的属性 

	- `fromValue` 

	- `toValue` 

	- `duration` 

- 用于调用动画的参数 

	- `timingFunction`  : 用来调整动画曲线的, 用这个函数操作曲线CAMediaTimingFunction()
	- `speed`: 动画的执行速度，0的时候动画就暂停了
	- `isRemovedOnCompletion` : 动画完成之后是否自动移除动画
	- `fillMode` : 动画在非active阶段的状态。 这个属性需要在 `isRemovedOnCompletion`为false的时候会生效。
		- `kCAFillModeRemoved` : 在动画开始之前和动画结束之后对layer没有影响，layer会处于没有添加动画的状态
		- `kCAFillModeForwards` : 动画结束之后，layer就保持动画最后的状态
		- `kCAFillModeBackwards` : 动画开始之前，layer只要添加了动画，layer就设置成了动画的fromValue的状态
		- `kCAFillModeBoth` : 上面两个的综合
	- `autoreveres` : 动画执行完了之后自动的反向再执行一遍

- 经常使用的`keyPath`

	- `transform.scale` : 缩放动画
	- `opacity` : 透明度。 *需要注意没有`alpha`这个keypath*

### case1: 通过CoreAnimation实现最基本的缩放动画 

```
    let scaleAnimation = CABasicAnimation(keyPath: "transform.scale")
	scaleAnimation.fromValue = 0
	scaleAnimation.toValue = 1
	scaleAnimation.duration = 1
	scaleAnimation.fillMode = .forwards
	scaleAnimation.isRemovedOnCompletion = false
	self.animationView.layer.add(scaleAnimation, forKey: nil)
```

### case4: 动画组的实现方式

### case2: 通过CoreAnimation实现 动画完成回调

通过设置动画的代理可以实现对动画开始和结束的回调处理

然后实现`animationDidStop`方法

```
	func scaleAnimation() {
		// 动画的设置
		let scaleAnimation = CABasicAnimation(keyPath: "transform.scale")
		scaleAnimation.fromValue = 0
		scaleAnimation.toValue = 1
		scaleAnimation.duration = 1
		scaleAnimation.fillMode = .forwards
		scaleAnimation.isRemovedOnCompletion = false
		scaleAnimation.delegate = self
		self.animationView.layer.add(scaleAnimation, forKey: nil)
	}

	func animationDidStop(_ anim: CAAnimation, finished flag: Bool) {   	    
    }
```

### case3: 动画暂停的实现

## UIView动画块

## UIView动画事务

## 仿射变换transform