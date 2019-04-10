# 代码规范 

统一的代码风格有助于提高开发效率。

定义统一的代码片段，能够快速书写代码，并且规范统一。

定义统一的方法名、代码分段，能够使开发人员快速定位代码位置。并且代码格式更加清晰

## 代码书写规范

### 代码分段

* `#pragma mark - Life Cycle`

	生命周期方法

	*加入到自定义代码片段中, 快捷键使用`mark_lifecycle`*

* `#pragma mark - Private Method`

	私有方法

	比如自定义视图的布局等

	*加入到自定义代码片段中, 快捷键使用`mark_privatemethod`*

* `#pragma mark - Public Method`

	对外提供方法

	*加入到自定义代码片段中, 快捷键使用`mark_publicmethod`*

* `#pragma mark - Event Action` 

	事件响应

	*加入到自定义代码片段中, 快捷键使用`mark_eventaction`*

* `#pragma mark - Delegate Datasource`

	代理方法

	*加入到自定义代码片段中, 快捷键使用`mark_delegate`*

* `#pragma mark - Getter Setter`

	懒加载方法

	*加入到自定义代码片段中, 快捷键使用`mark_getter`*

### 自定义代码段及快捷键

通过定义一些自定义代码片段，来保证每次对相同代码都是统一格式，第二可以减少代码书写量，通过代码片段直接生成

#### 属性的代码片段

* `prop_strong`

	```
    	@property (nonatomic, strong) <#NSString#> *<#str#>;
	```

### 自定义方法名

统一自定义方法名，方便查找相关代码。

* 自定义视图布局的方法名

	`layoutUserCreatedViews`

* 按钮事件响应的方法名

	`cancelBtnAction`

* RAC中信号处理方法

	`subscribeSignals`

## 通过模板统一代码格式

创建自定义模板
