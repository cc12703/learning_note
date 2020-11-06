[TOC]

# Electron框架

## 总体

* 基于Chromium和Node.js实现
* 使用前端技术来构建跨平台的桌面应用

## 原理

### 进程模型

* 应用分为主进程、渲染进程
* 主进程只有一个，管理所有的窗口和对应的渲染进程
* 渲染进程可以有多个，相互独立的

### 进程互访

#### 消息传递

* 使用ipcMain, ipcRenderer进行消息发送和接收


## 界面

### 窗口

* 由BrowserWindow类表示
* frame属性设置为false，可以屏蔽掉系统标题栏


### 页面

* webContents是核心模块，负责渲染和控制应用内的Web页面
* 每个webContents都有一个唯一ID
* webFrame 负责控制子页面

#### 页面加载事件

1. did-start-loading 开始加载
2. page-title-updated 页面标题更新
3. dom-ready 页面dom加载完成
4. did-frame-finish-load 框架加载完成
5. did-finish-load 当前页面加载完成
6. page-favicon-updated 页面icon图标更新
7. did-stop-loading 所有内容加载完成

#### 页面容器

* webFrame是最常见的页面容器，每个iframe对应一个webFrame实例
* mainFrame为主webFrame
* BrowserView 被设计成子窗口的形式



### 脚本注入

* 用于将一段js代码注入到目标网页中
* 该代码可以访问网页的任意内容、通过Node.js访问系统
* 通过webPreferences.preload来加载一个js文件
* 通过webContents.executeJavaScript来加载js代码


### 数据

#### 本地文件

可以使用app.getPath来获取不同的路径

* temp 临时文件夹
* appData 用户个性化数据目录
* userData 应用程序对应的个性化数据目录

#### 浏览器技术

##### Cookie 

* 只能存储少量数据，最多4KB
* 可以被服务端程序访问
* 数据带有有效期，到期后会被自动删除

##### LocalStorage

* 只能存储有限的数据，最多10MB
* 只能被客户端程序访问
* 无过期时间

##### SessionStorage

* 与Local Storage类似
* 浏览器关闭后，会被自动清空

##### IndexedDB

* 基于js的面向对象的数据库
* 无存储容量限制
* 三方库 Dexie.js，pouchdb, rxdb


#### SQLite

* 轻型的、嵌入式的SQL数据库引擎
* node-sqlite3 node.js的绑定库
* knexjs SQL指令构建器

### 性能

#### 全局异常

##### 捕获

* 利用Node.js技术来捕获
* 开发者工具中不会输出异常信息
* 可以在主进程和渲染进程中使用

```
process.on('uncaughtException', (err, origin) => {
    //收集日志
    //显示提示
})
```

#### 异常恢复

* 监听webContents的crasheds事件，发现渲染进程崩溃
* 监听webCotnents的unresponsive事件，发现界面未响应
* 显示对话框，提示用户重启或退出

```
win.webContents.on('crashed', async (e, killed) => {
    //收集日志
    //显示对话框
})
```



### 安全

#### 保护源码

##### 使用立即执行函数

* 不必为函数命名，避免污染全局变量
* 函数内部形成一个单独的作用域，外部代码无法访问

```
(function() {
    console.log('my func')
})()
```

##### 禁用开发者调试工具

设置webPreferences.devTools 为 false

##### 源码压缩、混淆

* webpack 压缩代码
* uglify-js  混淆代码

##### 使用asar

* 一个文件的归档格式
* 与tar格式类似
* 无法使用解压软件解压

##### 使用v8字节码

* 推荐bytenode
* 只用于保护关键的核心代码


## 实践

### 工具、库

* electron-builder 构建工具，用于构建和分发
* rxdb  实时NoSQL数据库
* vue-electron-builder vue-electron工程模板

### 代码树

```
    tsconfig.json   ts配置文件
    vue.config.js   vue配置文件
    package.json    webpack配置文件
    public/        资源目录
    src/
        background.ts  系统启动文件
        main.ts       界面启动文件
        common/    公共代码
           types/   ts数据类型定义
        main/      后台功能代码（运行在主进程）
        render/    界面代码（运行在渲染进程）
           App.vue  主界面
           components/  通用组件
           router/     路径定义
           store/      数据管理
           pages/      页面
        
```