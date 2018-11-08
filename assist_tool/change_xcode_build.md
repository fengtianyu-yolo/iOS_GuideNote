
# XCode10下使用原来的编译系统

当升级`XCode`到10版本之后，在`archive`时遇到报错`Cycle inside`

这是因为，`XCode10`下有了一套新的编译系统，并且是默认使用的。新的编译系统对工程有了更严格的检查。所以，会导致出现使用`XCode9`可以编译通过的工程，在`XCode10`下无法编译通过。

简单的解决方式可以使用如下的方式更改`XCode10`的编译系统。仍然使用旧版的编译系统。

1. 选择`File` - `workspace setting`

![xcode workspace setting](https://github.com/cocacola-ty/Images/blob/master/xcode_workspace_setting.png?raw=true)

2. 在弹出框中选择 `Legacy Build System`

![select build system](https://github.com/cocacola-ty/Images/blob/master/xcode_build_system.png?raw=true)

# `xcodebuild`使用旧版本编译系统

上面的方式只更改了手动通过`XCode`打包时编译系统。使用`xcodebuild`命令进行打包时的编译环境仍然使用的是最新的编译系统。

可以通过在`xcodebuild`命令后通过`-UseModernBuildSystem=<value>`参数来指定所使用的构建系统。`value=0` 或 `value=NO`表示使用旧版本构建系统(Legacy build system)。`value=1`或`value=YES`表示使用新的构建系统。

如下所示

```
 xcodebuild archive -workspace Toon.xcworkspace -scheme Toon -configuration Release -archivePath ~/Desktop/Toon.xcarchive CODE_SIGN_IDENTITY="iPhone Distribution: Beijing Syswin Internet Technology Co. Ltd." PROVISIONING_PROFILE="d73920ec-9178-4f26-a0b8-c89ca2bbea6d" -UseModernBuildSystem=NO
```

# 多`XCode`时，更改`xcodebuild`所使用的XCode版本

当安装了多个版本的`XCode`时，可以通过如下方式选择`xcodebuild`所使用的`xcode`版本的

打开`xcode`偏好设置，选择`location`

![xcodebuild select](https://github.com/cocacola-ty/Images/blob/master/xcode_command_line_tool_select.png?raw=true)

或 可以通过`xcode-select`命令进行选择

## 参考资料

http://www.skyfox.org/modify-legacy-or-new-build-system.html