
当工程使用CocoaPods进行第三方库及私有组件库的管理时，在开发时使用统一的组件库的版本很重要，可以保证所有成员的开发环境是一样的

## 实现方案

1. 选择`Build Phases`

2. 打开`[CP]Check Pods Manifest.lock`

## 原理

`[CP] Check Pods Manifest.lock` 选项的作用是在编译时执行一段脚本

这段脚本的作用是检查`Manifest.lock`和`Podfile.lock`的文件内容是否一致

当不一致时，中止编译，并输出错误

### Manifest.lock和Podfile.lock文件是什么

Manifest.lock文件是Podfile.lock文件的备份，文件中记录的当前各个Pod库所使用的版本等信息

Manifest.lock文件存放在`Pods/`文件夹中

通常情况下`Pods`是不会提交到git中进行管理，而Podfile.lock是会提交到git中用于保证所有开发者使用同一版本的库

当其他开发者的组件库版本进行了更新之后，会更新Podfile.lock并提交到git

而拉取代码之后，编译时比较Podfile.lock和Manifest.lock中存在不一致，表示本地Pods中的组件版本和其他开发者使用的组件版本不一样。需用`pod install`安装同样版本的组件库

可以看做`Manifest.lock`中记录了当前所使用各个组件库的版本，而Podfile.lock记录了其他开发者使用的库的版本。两者内容不同时则需要重新安装组件库

