# CocoaPods的使用

## 设置CocoaPods

### 安装 

CocoaPods需要Ruby进行构建，所以它会通过MacOS上默认的Ruby环境进行安装。

用户可以使用Ruby的版本管理工具来使用不同的Ruby环境，但是如果不是十分熟悉Ruby的版本管理工具，官方推荐使用默认的。

如果使用默认的Ruby环境安装`gems`的工具时，就需要使用`sudo`进行授权。

_`gem`是Ruby的包管理器，是Ruby标准库的一部分，可以直接通过gem来安装ruby的工具和第三方库等，如cocoapods就是ruby的一个工具_

```
	sudo gem install cocoapods
```

### 低权限的安装

如果不想授权`RubyGems`进程管理员权限。可以让`RubyGems`安装到指定的用户目录通过下面两种方式

1. 当执行`gem install`时指定`--user-install`参数

	```
		gem install cocoapods --user-install
	```

	即使是使用了`--user-install`，仍然需要配置`.bash_profile`文件的PATH变量。或者在使用时通过命令的全路径来使用。可以通过`gem which cocoapods`来查看命令所在的路径

	```
		$ gem which cocoapods 
		/User/fty/.gem/ruby/gems/2.6.0/gems/cocoapods-1.7.5/lib/cocoapods.rb
		$ /Users/fty/.gem/ruby/2.6.0/bin/pod install
	```

2. 通过设置`RubyGems`的环境变量 

	在`.bash_profile`添加下面两行内容 

	```
		export GEM_PATH=$HOME/.gem
		export PATH=$GEM_PATH/bin:$PATH
	```

第二种方式是官方推荐的最佳解决方案

### 升级

如果需要升级CocoaPods，直接再重新执行安装就行

```
$ [sudo] gem install cocoapods
```

如果安装的时候使用了`sudo`，升级的时候同样需要使用`sudo`

## 使用CocoaPods

