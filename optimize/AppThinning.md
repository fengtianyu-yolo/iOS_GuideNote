# 包大小优化

![包大小优化内容](https://upload-images.jianshu.io/upload_images/1284329-6390499ea74e3fad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## iOS中几种包的说明

* .app 

	编译之后的原始文件，实际是一个文件夹，里面包含所有的资源文件(图片、第三方bundle、plist文件等)、可执行文件、对所有文件的签名记录(`_CodeSignature`)

* .ipa

	就是讲.app放到Payload文件夹，然后对Payload文件夹进行zip压缩，最后修改了压缩文件的扩展名

* .xcarchive
	
	也是一个压缩包，将.ipa 和 .dSYM文件放到一起进行了压缩

* 从AppStore下载的包

	从AppStore下载的包就是IPA包，但是这个IPA包是经过苹果处理的

	**我们生成的IPA包上传到AppStore之后，苹果会对这个IPA文件进行解压缩，然后会其中的.app进行加密(Apple Fairlay DRM)处理。然后重新压缩成.ipa**

	用户最终下载的包，是这个经过重新处理之后的.ipa包

* appstoreconnect 后台的下载大小

	下载.ipa的

* appstoreconnect 后台的的安装大小

	安装大小是这个App安装之后占用磁盘的大小

	也就是说这个安装大小其实是.ipa进行解压之后的.app的大小。不过这个.app已经进行过加密处理了，所以会变大一些

## 各个优化内容的详细实践过程

### 如何查找未使用的图片资源

> 工具：LSUnusedResources

查找未使用的图片

1. 获取所有的图片名，放到一个集合中

2. 遍历每一个文件，检查文件中是否出现该图片的名，如果出现该图片名，则从集合中移除。

	1. 如果出现 `imageNamed:@""` 字符串，则表示该图片有使用

	2. 如果出现 `@""`，表示该图片可能是被使用，需要再检查

	3. 需要特殊注意带有规则的图片名 ， 如@`"img_%d"` 这种的

***

### 如何查找并删除重复的图片

> 工具：fdupes

比较图片的MD5，如果MD5是相同的则是相同图片。

***

### 如何查找相似的图片并删除

做图片的相似度比较，图片相似度高的，可以选择删除

***

### 如何查找没有使用的类文件

##### 方式一：逆向可执行文件

1. 通过otool逆向`__DATA._objc_classlist` 可以拿到所有的类列表

2. otool 逆向 `__DATA._objc_classrefs` 可以拿到使用的类的列表

3. 两个集合的差集就是未使用的类

##### 方式二：正则匹配去比较

1. 拿到所有的类文件的文件名

2. 去遍历每个文件，通过匹配特定字符串，如`[xx alloc]`、`[xx new]` 等字符串去筛选

##### 方式三：AppCode的Inspect Code 功能

***

### 如何查找代码中没有使用的方法

***

### 本地图片尝试使用webp格式的实践过程

1. 验证webp格式的图片体积比png格式的图片体积能缩小多少

2. 验证webp格式的图片在AppThinning的时候是否会有效

	不会

3. 验证webp格式的图片在打包的时候是否会走Xcode的压缩

	不会

**结论：对于放在bundle中的较大的图片可以尝试使用转成webp格式**

#### 工具准备

谷歌提供了png转webp的工具 

	* https://developers.google.com/speed/webp/docs/precompiled  
	* https://developers.google.com/speed/webp/docs/using

安装：用homebrew安装 `brew install webp`

基本用法：

* png 转 webp ：`cwebp -q 90 image.png -o image.webp` 

* webp装png :`dwebp image.webp -o image.png`

***

### 针对特定文件去优化

通过LinkMap统计占用空间较大的文件，去单独优化处理


## 参考链接

* https://zhuanlan.zhihu.com/p/19925959 

* https://www.jianshu.com/p/759f237c524a

