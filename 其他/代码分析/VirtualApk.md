# VirtualApk

[TOC]

## 特点
* 支持四大组件
* 组件无需站在你宿主中预注册
* 插件可以依赖访问宿主的代码和资源
* 系统兼容性高


## 原理
### 基本原理
* 运行时合并宿主和插件的class-looader
* 运行时合并宿主和插件的资源
* 构建时去除插件对宿主的引用（代码和资源）

### 四大组件
* Activity: 采用宿主manifest占坑方式来绕过系统校验，然后加载真正的activity
* Service: 动态代理AMS，拦截service相关请求，由代理Service分发操作
* Receiver: 将插件中的静态注册receiver重新动态注册一次
* ContentProvider: 动态代理IContentProvider，拦截provider相关请求，由代理Provider分发操作


## 代码分析
### activity
#### hook点
使用VAInstrumentation替换系统的Instrumentation，拦截以下点：
* 启动activity
* 加载activity
* 创建activity对象
* 调用onCreate函数

#### 启动activity
##### 位置 
* Instrumentation.execStartActivity函数

##### 逻辑
1. 转换隐式intent为显示，查找一遍插件来解析intent
2. 如果目标包名是插件的，转换intent为stub activity，给intent增加isPlugin标识
3. 调用真正的execStartActivity接口，启动intent


#### 加载acivity
##### 位置
* Instrumentation的LAUNCH_ACTIVITY消息

##### 逻辑
1. 如果intent来自插件，则更新theme为插件


#### 创建activity对象
##### 位置
* Instrumentation.newActivity函数

##### 逻辑
1. 正常加载类
2. 若成功，则调用真正的newActivity函数
3. 若失败，则从插件加载类
    1. 获取intent中的包名信息
    2. 使用包名查找到插件
    3. 调用真正的newActivity函数
    4. 设置activiy的mResources字段


#### 调用onCreate函数
##### 位置
* Instrumentation.callActivityOnCreate函数

##### 逻辑
1. 检查intent是否来自插件
2. 若是，则处理内部字段
    1. 设置base-context的mResources字段
    2. 设置activity的mApplication字段
    3. 设置activity父类的mBase字段
    4. 设置activity的屏幕旋转方向
 3. 调用真正的callActivityOnCreate函数


### service
#### 原理
* 由代理service来分发请求给对应的插件service
* 代理service分为LocalService, RemoteService

#### hook点
使用IActivityManager动态代理替换系统的AMS，拦截以下点：
* 启动service
* 停止service
* 绑定service

#### 启动service
##### 逻辑
1. 查找一遍插件来解析intent
2. 若解析失败，则直接调用系统接口
3. 若解析成功，则使用代理service启动service
     1. 转换intent加入代理service信息，额外命令EXTRA_COMMAND_START_SERVICE
     2. 调用代理context的startService接口

### 停止service
##### 逻辑
1. 查找一遍插件来解析intent
2. 若解析失败，则直接调用系统接口
3. 若解析成功，则使用代理service停止service
    1. 转换intent加入代理service信息，额外命令EXTRA_COMMAND_STOP_SERVICE
    2. 调用代理context的startService接口


### provider
#### 原理
* 由代理provider来分发请求给对应的插件provider

#### hook点
* 使用PluginContentResolver来替换掉系统的Resolver，拦截以下点：
    * 请求Provider
* 使用IContentProvider动态代理替换系统的provider，拦截对应的接口
* 包装uri成代理Provider的uri，转发操作给代理Provider (RemoteContentProvider)

#### RemoteContentProvider
##### 逻辑
1. 从uri中获取插件的包名和uri
2. 查找或生成插件的ContentProvider
3. 调用插件ContentProvider的对应函数



### 插件环境
#### PluginContext
##### 要点
* base为宿主context

##### 拦截点
1. getContentResolver  返回 PluginContentResolver
2. getPackageManager  返回 PluginPackageManager
3. getResources
4. getAssets
5. getTheme
6. startActivity


### 资源合并
#### 位置
* LoadedPlugin.createResources

#### 逻辑
1. 生成AssetManager加入宿主apk
2. 加入本插件apk 和 其他插件的apk
3. 生成Resources对象，适配系统的自定义Resources类
4. 替换其他插件的资源对象
5. 将Resources对象加入宿主中



## gradle插件

### 宿主工程
#### 功能
* 生成依赖库描述文件（VAHost/versions.txt）
* 保存资源R符号文件 （VAHost/Host_R.txt）
* 保存混淆map文件

### 插件工程
#### 功能
* 去除宿主上使用库
* 去除宿主的资源



## gradle插件代码分析

### VAPlugin
#### 任务hook机制
* 向gradle添加任务执行监听器
* 根据任务名字找到对应的hooker
* 在任务执行前调用hooker的beforeTaskExecute
* 在任务执行后调用hooker的afterExecute

#### hooker
##### PrepareDependenciesHooker 
* 监听 AppPreBuildTask 任务
* 根据host/versions.txt，生成需要裁剪的依赖列表（jar, aar）

##### MergeAssetsHooker 
* 监听 MergeSourceSetFolders 任务
* 去除要裁剪的aar中的assets目录

##### MergeManifestsHooker 
* 监听 MergeManifests 任务
* 去除要裁剪的aar中的manifest信息

##### MergeJniLibsHooker 
* 监听 TransformTask 任务
* 去除要裁剪的so文件

##### ProcessResourcesHooker 
* 监听 ProcessAndroidResources 任务
* 移除插件资源中宿主的资源
* 修改插件的资源ID

##### ProguardHooker 
* 监听 TransformTask 任务
* 在混淆时加入宿主的map文件

#### DxTaskHooker 
* 监听 TransformTask 任务
* 压缩R class文件大小
