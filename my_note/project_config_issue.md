# XCode工程配置导致的常见问题

* XCode - `Capabilities` 选项卡中一部分开关不显示

	这个选项卡下的开关是根据当前所选择的证书决定的。当证书中不支持的功能是不在该选项卡下显示的。

***

* Instruments 调试Debug模式的APP看不到具体的方法名

	因为默认情况下Debug模式是不会讲符号表文件`dSym`写入到APP中

	在`Build Setting` - `Debug Information Format` 选择 `DWARF with dSYM File` 

	再次调试就可以看到具体的方法名

*** 

* XCode运行时报错 `The run destination is not valid for runing `

	检查下面几个配置项

	1. `Deployment Target` 设备的版本和这个配置项选择的部署版本是否一样

	2. `Architectures` 架构选择的是否正确

*** 

