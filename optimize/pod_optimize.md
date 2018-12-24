# Pod库优化过程

当工程开发越来越大时，通过会进行组件化拆分。相关的代码作为一个模块。然后将该模块作为一个私有pod库来进行管理。

通过这样拆分可以使功能更加独立。再通过路由机制进行模块间解耦。

但是这样存在的问题是只是将代码分散到了各个私有Pod库中。对于工程的编译速度并没有提高。

对于需要开发的模块来讲，是不需要了解其他模块的源码的。所以，其他模块可以只提供一个framework，这样，对于某个模块的开发者来说，编译时只编译自己当前开发模块的代码。其他模块使用编译好的framework。会较大提高开发速度。

## 将Pod库打包成framework

### Pod打包framework的基本过程

1. 提交当前开完完成的代码。并打tag

2. 更改podspec的version。指向最新提交的tag

3. 通过pod package 对当前模块代码进行打包

    这里framework的构建会根据当前podspec中的version对应的版本

    打包完之后可以验证一下这个framework是否有问题。将这个pod库指向本地，运行工程看是否可以正常运行。

4. 将打包好的framework提交到git。打个tag

5. 更改podspec中的version。指向这个最新的含有framework版本的tag。

6. 提交podspec文件。

    ```
    pod repo push Spec库 xx.spec 
    ```

### framework和源码切换

通过上面的操作Pod库中的代码打包成了framework。

但是此时拉下来的Pod库仍然是源代码。

因为在pod库的podspec文件中仍然是指定的`source_files`属性，如下所示

```
s.source_files = "*.{.h,.m}"
```

`source_files`属性表明了需要获取仓库中的所有源代码。

要想更改为获取仓库的framework。需要指定`vendored_framework`属性。如下所示

```
s.ios.vendored_framework = "*.framework"
```

`vendored_framework`属性和`source_files`属性不可以同时存在，否则会导致重复文件

`s.ios`表示针对iOS平台。如果直接`s.vendored_framework`则是针对所有平台

**所以，根据`vendored_framework`和`source_files`属性可以决定当前Pod库是提供的源码还是framework。**

但是，当开发时会有源码和framework切换使用的情况存在。

这种情况下，可以在podspec中添加一个判断条件，根据命令行中传入的参数，决定使用哪种模式。如下所示。

```
if ENV['use_source'] or ENV[s.name + 'use_source']
    s.source_files = ''
else 
    s.vendored_framework = 'Framework/**/*.framework'
```

-`Framework/**/*.framework`中的`**`代表递归所有`Framework`文件夹下的子目录中的`framework`-

这样当使用的时候`pod install`就会拉取Pod库中的`framework`。

而当带有如下参数时则拉取Pod中的源码

```
use_source=1 pod install
```

在源码和framework切换的时候如果出现拉取到一个空文件夹时，删除一下缓存再重新尝试 `pod cache clean --all`。并且同时删除`Pods`文件夹中的该模块

*如果经常需要进行源码和framework的切换时，可以通过Podfile中`pre_install`这个hook来做缓存清理的操作*

### pod package 进行打包的详细说明

```
pod package MyPod.podspec --exclude-deps --no-mangle --spec-sources=git@gitlab.com:MyRep/MyPod.git,https://github.com/CocoaPods/Specs.git
```

#### 参数说明

* `--exclude-deps`
    
    打包的时候不将依赖的库打包进去。
    
    不加该参数默认的会将该Pod库所依赖的库也打包进这个framework中。

* `--no-mangle`  

* `--spec-sources`

    spec仓库地址

    需要将自己的私有库地址放在前面，这样先去私有的spec仓库中去找podspec文件

## 遇到的问题

### 1. 推spec时，xcodebuild使用的最新的构建系统，编译不通过

### 2. 改用framework后头文件要改用<>方式引用

将Pod库打包成framework之后，我们需要使用该Pod库中的文件时，需要通过尖括号的引用方式`import <xx/xx.h>`来导入头文件，否则会报找不到头文件的错误。

如果之前是使用的源码方式，切换更改为framework方式之后要将所有的引用方式更改为尖括号的引用方式。

如果仍然想使用双引号的引用方式。可以按照如下方式添加配置

在Podfile中添加下面的代码

```
post_install do |installer|

    installer.pods_project.targets.each do |target|

        project.build_configurations.each do |config|
            Pod::UI.puts "add configuration..."
            target.build_settings['FRAMEWORK_SEARCH_PATHS'] = "${SRCROOT}/**"
            target.build_settings['HEADER_SEARCH_PATHS'] = "${SRCROOT}/**"
        end

    end
end
```

这段代码的作用是在执行`pod install`之后，在每个Pod的target中的`build setttings`中的`framework search path`和`header search path`选项添加`${SRCROOT}/**`

即如图所示的位置

![pod-target-setting](https://github.com/cocacola-ty/Images/blob/master/pod_target_build_setting.png?raw=true)

上面代码中的`FRAMEWORK_SEARCH_PATHS`字段和`HEADER_SEARCH_PATHS`这两个字段名可以在`Pods/Pods.xcodeproj/project.pbxproj`中找到。

`${SRCROOT}/**` 中的`${SRCROOT}`指的是工程的根目录也就是和xx.xcodeproj同级的目录 。`/**`则是指下面的所有子目录

还可能会用的变量有`${PODS_ROOT}`,这个变量的定义是`${SRCROOT}/Pods`。也就是`Pods`文件夹位置

## 实现代码提交后自动完成打包过程

要实现代码提交之后自动打framework的流程，首先要实现可以通过脚本来完成上述的打framework的过程。

### 脚本完成打framework

1. 首先要校验podspec文件是否有符合规则

    这里的规则是指当前podspec文件是否添加了判断条件，什么时候使用源码什么情况下使用framework

    读取podspec文件，检查是否有一行可以匹配`if ENV[s.name + 'use_code']`。通过检查文件中是否有这个判断条件来认为是否已经添加

    如果没有这行内容，则认为没有这个判断。查找文件中的`s.source_file`,替换为判断条件

2. 更改当前podspec的version

    首先通过`git tag | tail -1` 获取当前的tag

    通过字符串处理，将tag的最后一位增加1

    以该字符串作为最新的version

    修改过程：

    首先将当前读取的行去掉所有空格，检查是否有`version=`内容

    通过正则表达式替换`""`或`''`之间的内容

3. 记录上一步的version。对当前代码按照version打tag。

    这样，实现了podspec中指向了最新提交的代码

4. 调用pod package命令进行打包

5. 移动framework的所在文件夹

6. 删除构建过程产生的文件夹

7. 提交framework

8. 更改podspec的version，指向提交framework之后的tag