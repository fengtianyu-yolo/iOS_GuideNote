# 自定义的代码片段

## 属性的代码片段

* 强引用属性

	快捷键： `prop_strong`

	Title : Property Strong Type

	```
	@property (nonatomic, strong) <#NSString#> *<#str#>;
	```

* block属性

	快捷键： `prop_block`

	Title : Property Block Type

	```
	@property (nonatomic, copy) void (^<#blockName#>)(void);
	```

* 基本数据类型属性

	快捷键： `prop_assign`

	Title : Property Assign Type

	```
	@property (nonatomic, assign) <#NSInteger#> <#integer#>;
	```	

***

## Mark的代码片段

* 声明周期代码段

	快捷键 ：`mark_lifecycle`

	Title : Mark Life Cycle

	```
	#pragma mark - Life Cycle
	```

* 初始化方法代码段

	快捷键 ：`mark_initmethod`

	Title : Mark Init Method

	```
	#pragma mark - Init Method
	```

* 私有方法代码段

	快捷键 ：`mark_privatemethod`

	Title : Mark Private Method

	```
	#pragma mark - Private Method
	```

* 对外方法代码段

	快捷键 ：`mark_publicmethod`

	Title : Mark Public Method

	```
	#pragma mark - Public Method
	```

* 事件响应代码段

	快捷键 ：`mark_eventaction`

	Title : Mark Event Action

	```
	#pragma mark - Event Action 
	```

* 代理方法代码段

	快捷键 ：`mark_delegate`

	Title : Mark Delegate DataSource

	```
	#pragma mark - Delegate DataSource
	```

* 懒加载方法代码段

	快捷键 ：`mark_getter`

	Title : Mark Getter Setter

	```
	#pragma mark - Getter Setter
	```
	
***

## 定义静态常量的代码片段

* 字符串类型静态变量

	快捷键 ： `static_string`

	Title : Static String

	```
	static NSString const * <#name#> = @"<#string#>";
	```

* `CGFloat`类型静态变量

	快捷键 ： `static_cgfloat`

	Title : Static CGFloat

	```
	static CGFloat const <#name#> = <#10.0#>;
	```

* `NSInteger`类型静态变量

	快捷键 ： `static_nsinteger`

	Title : Static NSInteger

	```
	static NSInteger const <#name#> = <#10#>;
	```
