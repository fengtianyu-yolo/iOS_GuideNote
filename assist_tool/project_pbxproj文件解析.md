
`project.pbxproj`文件是`XCode`工程的配置文件。该文件为旧版本的plist格式内容

大概样式如下

```
objects = {
    /* Begin PBXBuildFile section */
    2150742CBE6728E47D2531A0 /* libPods-TLauncherTests.a in Frameworks */ = {isa = PBXBuildFile; fileRef = 32F7DB5B5BB30F7773A967F9 /* libPods-TLauncherTests.a */; };
}
```

## 大概分为以下几类信息

* 工程中的文件关联信息、资源关联信息

	* `PBXBuildFile` 参与编译的文件

	* `PBXFileReference` 工程中的所有文件信息

* 文件的组织结构信息

	* `PBXGroup` 工程中的文件夹

* 工程的编译配置、证书配置信息
	
	* `PBXResourcesBuildPhase` 编译阶段的资源配置

	* `PBXFrameworksBuildPhase` 编译阶段的`framework`配置

	* `PBXProject` 工程信息

	* `PBXNativeTarget` 工程中所有target的信息

	* `XCConfigurationList` 每个target下包含的编译模式,如`Debug` 、 `Release`等模式

	* `XCBuildConfiguration` 具体的编译信息,如`Release`模式下的编译配置

每一项资源在这个文件中都有一个值作为唯一标识，如`903C829A2075C24300EB9AD0`。可以看做是id。

一般每个ID值后面都会有一个注释来进行说明这个ID对应的具体内容。如`903C829C2075C24300EB9AD0 /* TLauncher */`,表示这个ID是代表的是`TLauncher`这个`target`。

相同类型的资源是按段进行整理的 

每一段内容前用`/* Begin xxx section */`注释作为开始。用`/* End xxx section */` 作为这一段内容的结束

每一项内容中用`isa = xxx` 指示该资源所属于的类型

示例如下 

```
    /* Begin PBXGroup section */
		694F336321538CE3008A83CF /* CustomModule */ = {
			isa = PBXGroup;
			children = (
				692930B821550DA80025C5A7 /* TNBluetoothJumpModel.h */,
				692930B921550DA80025C5A7 /* TNBluetoothJumpModel.m */,
			);
			path = CustomModule;
			sourceTree = "<group>";
		};
   /* End PBXGroup section */
```
第一行` /* Begin PBXGroup section */` 表示下面的内容是group相关的

第二行 `694F336321538CE3008A83CF /* CustomModule */ ` 表示 这个ID代表的是`CustomModule`这个文件夹资源， 下面的内容是这个文件夹中的详细内容

`isa = PBXGroup`表示这个资源属于`PBXGroup`类型

`children`属性表示该文件夹下所有的文件，值为每个文件的ID。

最后用`/* End PBXGroup section */` 表示 关于group相关的内容全部完成。

## 详细段落内容如下 

### `PBXBuildFile`

参与编译的文件

### `PBXContainerItemProxy`
### `PBXFileReference`

项目中的文件资源，包括源文件、app、framework等

文件引用相关内容

### `PBXGroup` 

工程中的文件夹相关信息

示例如下 

```
    /* Begin PBXGroup section */
		694F336321538CE3008A83CF /* CustomModule */ = {
			isa = PBXGroup;
			children = (
				692930B821550DA80025C5A7 /* TNBluetoothJumpModel.h */,
				692930B921550DA80025C5A7 /* TNBluetoothJumpModel.m */,
			);
			path = CustomModule;
			sourceTree = "<group>";
		};
   /* End PBXGroup section */
```

`Begin PBXGroup section` 用注释来作为这段内容的开始

`694F336321538CE3008A83CF` 作为这个文件夹的资源标识

`children` 下面放的该文件夹中的所有文件的唯一标识。并用注释标明具体的文件。

### `PBXNativeTarget`

这一段是工程中所有`target`的相关内容

```
    /* Begin PBXNativeTarget section */
		903C829C2075C24300EB9AD0 /* TLauncher */ = {
			isa = PBXNativeTarget;
			buildConfigurationList = 903C82B32075C24600EB9AD0 /* Build configuration list for PBXNativeTarget "TLauncher" */;
			buildPhases = (
				63595572631C95F927286EAF /* [CP] Check Pods Manifest.lock */,
				903C82992075C24300EB9AD0 /* Sources */,
				903C829A2075C24300EB9AD0 /* Frameworks */,
				903C829B2075C24300EB9AD0 /* Resources */,
				E6369FB196A4BD1A8A8BEB4A /* [CP] Copy Pods Resources */,
				99A4370ADE41D6A2380B14D8 /* [CP] Embed Pods Frameworks */,
			);
			buildRules = (
			);
			dependencies = (
			);
			name = TLauncher;
			productName = TLauncher;
			productReference = 903C829D2075C24300EB9AD0 /* TLauncher.app */;
			productType = "com.apple.product-type.application";
		};
    /* End PBXNativeTarget section */
```

 用`903C829C2075C24300EB9AD0`这个ID就代表了`TLauncher`这个`target`

 `buildConfigurationList`字段指明，`TLauncher`这个`target`的编译配置列表在`903C82B32075C24600EB9AD0`这个ID对应的内容下。在这个ID的对象下就可以找到当前target的`Debug`、`Release`等模式下的详细编译配置内容

### `PBXProject`

该工程的信息内容。

```
/* Begin PBXProject section */
		903C82952075C24300EB9AD0 /* Project object */ = {
			isa = PBXProject;
			...
			projectRoot = "";
			targets = (
				903C829C2075C24300EB9AD0 /* TLauncher */,
				F6886DD32086086600E80092 /* TLauncherTests */,
			);
			...
		};
/* End PBXProject section */
```

比如有`targets`字段包含有该工程下的所有target的信息

### `PBXFrameworksBuildPhase` 、`PBXResourcesBuildPhase`、`PBXShellScriptBuildPhase`、`PBXTargetDependency`、`PBXSourcesBuildPhase`

这几项内容都是对工程在编译阶段的配置

对应于工程中如下页面的配置内容

![build phase](https://github.com/cocacola-ty/Images/blob/master/build_phase.jpg?raw=true)

### `XCConfigurationList`

```
    /* Begin XCConfigurationList section */
		903C82B32075C24600EB9AD0 /* Build configuration list for PBXNativeTarget "TLauncher" */ = {
			isa = XCConfigurationList;
			buildConfigurations = (
				903C82B42075C24600EB9AD0 /* Debug */,
				903C82B52075C24600EB9AD0 /* Release */,
			);
			defaultConfigurationIsVisible = 0;
			defaultConfigurationName = Release;
		};
    /* End XCConfigurationList section */
```

这段中的每个ID对应的内容为某个`target`下的编译模式列表

如，`903C82B32075C24600EB9AD0`这个ID对应的内容是`TLauncher`这个target下的编译模式列表

`buildConfigurations`字段指明了这个target下的所有编译模式

比如`903C82B52075C24600EB9AD0`编号就代表Release模式下的编译配置。

通过这个值就可以在`XCBuildConfiguration`编译配置内容段查到详细的编译配置信息

### `XCBuildConfiguration`

这段中的内容是详细的编译配置信息

如下所示
```
903C82B52075C24600EB9AD0 /* Release */ = {
			isa = XCBuildConfiguration;
			baseConfigurationReference = 613FB9E3814AE8F26EA310B4 /* Pods-TLauncher.release.xcconfig */;
			buildSettings = {
				ALWAYS_EMBED_SWIFT_STANDARD_LIBRARIES = YES;
				ASSETCATALOG_COMPILER_APPICON_NAME = AppIcon;
				ASSETCATALOG_COMPILER_LAUNCHIMAGE_NAME = LaunchImage;
				CLANG_ALLOW_NON_MODULAR_INCLUDES_IN_FRAMEWORK_MODULES = YES;
				CODE_SIGN_ENTITLEMENTS = TLauncher/TLauncher.entitlements;
				CODE_SIGN_IDENTITY = "iPhone Distribution";
				CODE_SIGN_STYLE = Manual;
				DEVELOPMENT_TEAM = TLWP697Z5D;
				ENABLE_BITCODE = NO;
				GCC_PRECOMPILE_PREFIX_HEADER = YES;
				GCC_PREFIX_HEADER = TLauncher/TLauncher.pch;
				INFOPLIST_FILE = TLauncher/Info.plist;
				IPHONEOS_DEPLOYMENT_TARGET = 8.0;
				LD_RUNPATH_SEARCH_PATHS = (
					"$(inherited)",
					"@executable_path/Frameworks",
				);
				OTHER_LDFLAGS = (
					"$(inherited)",
					"-ObjC",
				);
				PRODUCT_BUNDLE_IDENTIFIER = com.systoon.enterprise.huairoutong;
				PRODUCT_NAME = "$(TARGET_NAME)";
				PROVISIONING_PROFILE = "92003ec7-f84e-4214-b27d-d47767f267b6";
				PROVISIONING_PROFILE_SPECIFIER = HuaiRouTong_Distribution;
				TARGETED_DEVICE_FAMILY = 1;
				USER_HEADER_SEARCH_PATHS = (
					"\"${PROJECT_DIR}/../Pods\"/**",
					...
				);
			};
			name = Release;
		};
```

这段中的每一个ID对应的内容就是某一target下指定模式下的详细编译配置。比如`TLauncher`target下`Release`模式的编译配置

包含证书配置信息等

`CODE_SIGN_STYLE` 签名方式

`DEVELOPMENT_TEAM` 开发者证书的team

`ENABLE_BITCODE` 是否打开`bitcode`

`PRODUCT_BUNDLE_IDENTIFIER ` 该模式下的`bundle_id`

`PRODUCT_NAME` 该模式下所使用的名称

`PROVISIONING_PROFILE` 该模式下所使用的描述文件

`PROVISIONING_PROFILE_SPECIFIER` 该模式下的描述文件的名称

## 应用场景

当通过脚本进行自动打包时，必然会涉及修改证书及`bundleId`。就可以通过修改该文件实现修改证书配置

大概思路如下。

1. 首先，通过字符串匹配`/* Begin PBXProject section */`，找到工程信息内容

2. 在该段内容中，通过查找`targets`字段。找到该工程下的所有`target`。通过字符串与注释进行匹配，找到我们需要打包的`target`的ID值

3. 获取到目标`target`的ID值之后。通过注释，筛选出`PBXNativeTarget`内容段。

4. 在`PBXNativeTarget`内容段中，找到`target`ID对应的内容。在这段内容中找到`buildConfigurationList`字段，获取到这个`target`下的编译配置列表对象ID。

5. 此时，进入`XCConfigurationList`内容段。在该范围中，通过上一步中的编译配置列表对象ID，获取该`target`下的配置模式列表内容。

6. 在配置列表中，找到需要使用的模式。拿到该模式对应的配置对象ID值。

7. 进入`XCBuildConfiguration`编译配置段。通过上一步中的对象ID值，定位到该配置对象。

8. 修改该配置对象中的内容。如可以通过修改`PROVISIONING_PROFILE`等字段的值实现更改证书。

