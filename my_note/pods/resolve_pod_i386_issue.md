# Pod库lint时报错不支持i386架构的几种解决方案

## 1 podspec中指定支持的架构

在podspec文件中添加

```
	s.pod_target_xcconfig = {
    	'VALID_ARCHS[sdk=iphonesimulator*]' => 'x86_64'
  	}
```

## 2 push podsepc时不进行校验

执行push的时候添加`--skip-import-validation`参数

如下所示

```
ᐅ pod repo push myspecs MyPod.podspec --use-libraries --sources=git@git.gitlab.com:native/MySpecs.git,git@github.com:CocoaPods/Specs.git --allow-warnings --verbose --skip-import-validation
```

## 3 修改rb文件

https://www.jianshu.com/p/88180b4d2ab7