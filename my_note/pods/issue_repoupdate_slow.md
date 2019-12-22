# `pod repo update`慢的解决方案

当执行`pod update` 时会先更新本地的Specs(/.cocoapods/repo/)仓库

会发现在更新过程中网速很慢，一般在10k左右，但是网速并不慢，而是通过git去访问会慢。

## 解决

给git设置代理

1. 打开翻墙软件。翻墙软件的默认端口是1087。可以去翻墙软件的设置中查看

2. 在终端中输入

	```
		git config --global http.https://github.com.proxy socks5://127.0.0.1:1087
		git config --global https.https://github.com.proxy socks5://127.0.0.1:1087
	```

3. 如果要取消上面的代理

	```
		git config --global --unset http.https://github.com.proxy 
		git config --global --unset https.https://github.com.proxy 
	```