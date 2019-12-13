# 启动速度优化

## 完整的启动过程

## 测量启动各阶段的用时

### 如何让测试数据更准确

断网或者使用飞行模式 

### Pre-Main阶段的查看

1. 通过`XCode` -> `Edit Scheme` -> `Enviroment Variables` 添加 `DYLD_PRINT_STATISTICS`值为1

2. 通过 `Instruments` -> `App Launch` 查看

### Main阶段的查看

## 如何优化

### Pre-Main阶段的优化

#### 如何定位到哪些load方法耗时较长

`App Launch` -> `Static Runtime Initialzer`

### Main阶段的优化

## 验证优化效果

## 相关工具的使用

## 查找耗时点

将启动过程划分为下面几个阶段，先去解决耗时高的阶段

1. Pre-Main 

2. didLaunchingWithOptions

3. 渲染

## 验证是否有效

## Link

https://mp.weixin.qq.com/s/Kf3EbDIUuf0aWVT-UCEmbA
