# 私有库中引用静态库

当私有Pod库中使用到了静态库，需要再`podspec`文件中将静态库引用进来。如下所示

```
	s.vendored_library = ‘MyPod/*.a’  

	s.vendored_framework = ‘MyPod/*.framework’

```

**需要注意的.a的名字需要以lib来开头**

如：`libAModule.a`
