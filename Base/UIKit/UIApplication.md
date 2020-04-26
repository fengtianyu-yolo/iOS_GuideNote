# UIApplication 

## 生命周期

- `- application: willFinishLaunchingWithOptions:`

	- App即将完成启动

- `- application: didFinishLaunchingWithOptions`

	- App已经完成了启动

- `applicationWillEnterForeground`

	- App将要进入前台状态

	- 场景：App处于后台时，点击App Icon，重新回到前台状态

	- 注意：**如果是App是启动，不会走这个方法**

- `applicationDidBecomeActive`

	- App进入活跃状态

	- 一般流程是先进入前台，然后才进入活跃状态

- `applicationWillResignActive`

	- App将要释放活跃状态

	- 场景：下拉通知栏、来电话、锁屏、双击Home键等。

	- 释放活跃状态后不一定就是进入后台了，比如下拉通知栏。这表示它只是不活跃状态了，有其他进程在操作

- `applicationdidEnterBackground`
	
	- App已经进入后台

	- 场景：点击Home键将App进入后台、锁屏. *从App中通过通用跳转进入其他App?*

	- **App进入后台之前一定会先调用`ResignActive`释放活跃状态。**

- `applicationWillTerminate`

	- App将要结束

- `applicationDidFinishLaunching`
