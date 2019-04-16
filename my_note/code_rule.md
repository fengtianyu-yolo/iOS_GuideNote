# 代码规范 

统一的代码风格有助于提高开发效率。

定义统一的代码片段，能够快速书写代码，并且规范统一。

定义统一的方法名、代码分段，能够使开发人员快速定位代码位置。并且代码格式更加清晰

## 代码书写规范

### 代码分段

* `#pragma mark - Life Cycle`

	生命周期方法

* `#pragma mark - Private Method`

	私有方法

	比如自定义视图的布局等

* `#pragma mark - Public Method`

	对外提供方法

* `#pragma mark - Event Action` 

	事件响应

* `#pragma mark - Delegate Datasource`

	代理方法

* `#pragma mark - Getter Setter`

	懒加载方法

### 使用统一的自定义代码片段

统一的代码片段的作用：

1. 生成的代码格式是统一的

2. 提高效率，减少重复代码的书写

### 统一的自定义命名

统一自定义方法名和属性名，方便查找相关代码。

#### 自定义方法名

* 自定义视图布局的方法名

	`layoutUserCreatedViews`

* 按钮事件响应的方法名

	`cancelBtnAction`

* RAC中信号处理方法

	`subscribeSignals`

#### 自定义变量名

* 说明类的Label 

	`textLabel`

	`explainLabel`

## 通过模板统一代码格式

通过模板使得创建出来的文件已经将上面的格式规范好

创建自定义模板
