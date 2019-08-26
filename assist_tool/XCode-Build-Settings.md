# XCode Build Settings

## Build Options 

* `Debug Information Format` :

	默认为`DWARF`,这样在`Debug`模式下，不会将`dSYM`文件写入到App中

	`DWARF with dSYM File` ： 选择该选项在`Debug`模式下也会将`dSYM`文件写入到App。方便使用`Instrment`等工具直接调试`Debug`的APP

## Search Paths

* `Header Search Paths` : 头文件的搜索路径

	当`import`头文件时，会根据这个选项配置的搜索路径去查找有没有这些目录下有没有要导入的头文件。

	当使用`CocoaPods`时，这个选项一般默认为`$(inherited)`

	