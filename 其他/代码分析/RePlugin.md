
# RePlugin

[TOC]


## 特点
* 支持四大组件
* 组件无需站在你宿主中预注册
* hook点只有一处（ClassLoader）
* 支持多进程运行插件


## 工程说明
* replugin-host-gradle  宿主Gradle插件
* replugin-host-library 宿主库
* replugin-plugin-gradle   插件Gradle插件
* replugin-plugin-libraray  插件库

### replugin-host-gradle
#### 作用
1. 动态修改manifest，加入占位声明
2. 动态生成 HostBuildConfig类
3. 扫描assets\plugins目录，生成plugins-builtin.json文件

### replugin-plugin-gradle
### 作用
1. 修改插件代码中的类继承：Activity
2. 修改插件代码中的部分系统类调用
    

## hook点
* 使用RepluginClassLoader替换了宿主的ClassLoader

### RepluginClassLoader实现
1. 继承PathClassLoader
2. 拷贝原始classLoader中的关键字段。欺骗系统
3. 使用反射方式调用原始classLoader中的重要方法。（findResource, findLibrary, getPackage）
4. 重写loadClass方法，优先从插件中加载类

### RepluginClassLoader替换
#### 方法
* PatchClassLoaderUtils.patch()

#### 逻辑
1. 获取宿主的base-context
2. 获取mPackageInfo字段（LoadedApk类）
3. 获取mClassLoader字段
4. 获取到原始的classLoader，并创建RepluginClassLoader
5. 将RepluginClassLoader设置给mClassLoader字段和Thead中的context-class-loader


## 多进程
* UI进程：宿主的主进程
* 常驻进程(persistent)：一个，服务器进行，用于插件管理
* 插件进程：多个，运行插件


## 关键类
### RePlugin  
* 框架的对接接口类
* 宿主用于操作插件

### RePluginApplication  
* 宿主application继承该类，用于初始化框架

### PmBase  
* 插件管理接口

### PluginManagerServer  
* 插件管理器
* 实现IPluginManagerServer接口
* 运行在常驻进程
* 功能：安装插件，卸载插件，获取插件，查询插件是否运行

### PluginContainers  
* 插件容器管理中心
* 功能：管理坑位

### PmHostSvc   
* 实现IPluginHost接口
* 运行在常驻进程

### PluginProcessPer 
* 实现IPluginClient接口
* 运行在所有进程

### PluginServiceServer  
* 插件service管理器
* 实现IPluginServiceServer接口
* 运行在所有进程




## 流程分析
### 框架初始化

**在宿主application初始化时进行**

#### RePlugin.App.attachBaseContext()
##### 逻辑
1. 初始化默认配置  ---> sConfig.initDefaults(app) 
2. 初始化IPC: 判断当前线程属于哪个类型  ---> IPC.init()
3. 初始化宿主配置 ---> HostConfigHelper.init()
4. 初始化框架 ---> PMF.init()
5. 加载框架 ---> PMF.callAttach()

#### PMF.init()
##### 逻辑
1. 初始化插件  ---> PmBase.init()
2. hook宿主的classLoader   ---> PatchClassLoaderUtils.patch(application)


### 插件安装
#### RePlugin.install()
##### 逻辑
1. 检查文件是否存在
2. 让常驻进程安装插件 ---> IPluginHost.pluginDownloaded()
3. 调用回调onInstallPluginSucceed

#### IPluginHost.pluginDownloaded() 
##### 逻辑
 1. 调用IPluginManagerServer.install()
 2. 更新常驻进程的插件表  PmBase.newPluginFound()
 3. 通知其他进程有插件被安装  ---> IPC.sendLocalBroadcast2AllSync()

#### IPluginManagerServer.install()
##### 逻辑
1. 从apk中获取内容PackageInfo
2. 校验apk的签名信息（可选）
3. 获取插件名字，版本号，协议版本号 ---> PluginInfo.parseFromPackageInfo()
4. 若插件已存在，则检查版本是否可以安装
5. 拷贝apk文件到指定目录
6. 释放apk中的so  ---> PluginNativeLibsHelper.install()
7. 刷新信息并保存


### 启动插件activity
#### RePlugin.startActivity()
##### 逻辑
1. 调用PluginLibraryInternalProxy.startActivity()
1. 若插件不存在，则调回调onPluginNotExistsForActivity
2. 处理动态映射类
3. 检查插件状态，若不正常，则调回调onPluginNotExistsForActivity
4. 若插件未释放则是打插件，则调回调onLoadLargePluginForActivity
5. 转换成坑activity  ---> PluginCommImpl.loadPluginActivity() 
6. 启动intent, 并调回调onPrepareStartPitActivity

#### PluginCommImpl.loadPluginActivity()
##### 逻辑
1. 加载插件，获取activity信息  --->  getActivityInfo()
2. 保存activity主题到intent中
3. 根据进程名选择进程号
4. 启动指定进程 ---> IPluginHost.startPluginProcess() 返回 IPluginClient接口
5. 分配该进程的坑位  ---> IPluginClient.allocActivityContainer()
6. 向intent中写入坑位信息


### 加载插件类
#### PMF.loadClass()  ---> PmBase.loadClass()
##### 逻辑
1. 如果是PluginPitService类名，则直接返回该类
2. 如果是activity坑位，则查找对应的插件activity类   ---> PluginProcessPer.resolveActivityClass()
3. 如果是service坑位，则只加载默认插件的service类
4. 如果是provider坑位，则只加载默认插件的provider类
