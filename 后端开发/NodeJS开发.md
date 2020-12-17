

# NodeJS开发


## 总体

* 一个强制不共享任何资源的单线程、单进程系统
* 目标是成为一个构建快速、可伸缩的网络应用平台

### 特点
* 异步IO：绝大多数的操作都是以异步的方式进行调用
* 事件：通过异步IO，将事件点暴露给业务逻辑
* 单线程：js代码运行在一个线程中，和其他线程是无法共享状态的
* 跨平台：基于libuv库来实现跨平台


## 模块机制

### CommonJS规范

#### 模块引用
使用require()接口

```js
var math = require('math');
```

#### 模块定义
使用exports对象

```js
exports.add = function() {
    //code
}
```

#### 模块标识
传入require的参数

* 符合小驼峰命名的字符串
* 相对路径、绝对路径
* 可以没有文件后缀


### node实现

#### 模块分类
* 核心模块：Node提供的模块，编译进行二进制执行文件中
* 文件模块：用户编写的模块，在运行时动态加载的

#### 加载过程
1. 检查缓存中是否存在加载过的模块
2. 分析路径，解析模块标识符
3. 定位文件，依次尝试.js, .node, .json后缀
4. 编译执行，根据不同后缀来处理

##### 模块标识
* 核心模块，像fs, http, path
* 相对路径文件模块，以 . 或 .. 开始
* 绝对路径文件模块， 以 / 开始
* 自定义模块

##### 模块加载
* .js文件：同步读取后编译执行
* .node文件：通过dlopen()加载
* .json文件：同步读取后，使用JSON.parse()解析


## 包机制

### CommonJS规范

#### 包结构
**本质是一个压缩包，zip或者tar.gz**

* package.json 包描述文件
* bin 存放可执行的二进制文件
* lib 存放代码
* doc 存放文档
* test 存放测试代码

#### 包描述文件
* name 包名
* description 简介
* version 版本号，格式 major.minor.revision
* dependencies 依赖包列表

### node实现
npm实现了该规范，作为node的包管理器

#### 增加的字段
* author 包作者
* bin 命令行信息
* main 包的模块入口
* devDependencies 开发时用的依赖包列表



## 异步IO

* linux上使用线程池来实现
* windows上使用IOCP来实现

**图示**
![](https://gitee.com/cc12703/figurebed/raw/master/img/20201106225005.png)


### 基本要素

* 事件循环
* 观察者
* 请求对象
* IO线程池


### 事件循环

Node自身的执行模型，一个无限循环，每次循环为一个Tick

* 检查是否有事件待处理
* 取出事件和对应的回调函数
* 执行回调函数

**图示**
![](https://gitee.com/cc12703/figurebed/raw/master/img/20201106225122.png)


### 其他异步API

#### setTimeout

* 原理：创建定时器对象，插入红黑树中
* 问题：精确度不高

**图示**
![](https://gitee.com/cc12703/figurebed/raw/master/img/20201106225148.png)



#### nextTick
* 原理：将回调放入队列，下一次tick时取出执行


### 网络IO

* 将socket上侦听到的请求生成事件交给IO观察者
* 事件循环不停的处理这些网络IO事件

**图示**
![](https://gitee.com/cc12703/figurebed/raw/master/img/20201106225214.png)


## 异步编程

* 优势：非阻塞IO可以使CPU与IO不相互依赖等待
* 缺点：事件循环线程无法承担太多，否则会影响任务调度

### 难点

* 异常处理
* 函数嵌套过深
* 容易阻塞代码
* 多线程编程
* 异步转同步


### 解决方案

#### 事件发布/订阅模式
* 是一种回调函数的事件化
* node提供了events模块
* 常常用来解耦业务逻辑

**继承events**
```
var events = require('events');

function Stream() {
    events.EventEmitter.call(this);
}
util.inherits(Stream, events.EventEmitter);
```

#### Promise/Deferred模式
已经发布在CommonJS规范中，包括Promises/A, Promises/B, Promises/D等模式


##### Promises/A
* Promise操作只会有三种状态：未完成、完成、失败
* Promise状态只能从 未完成，转化到 完成、失败
* Promise状态一旦转化，将不能被更改
* Promise对象只需要then()一个方法

###### then()要求
* 接受完成、错误的回调方法，在状态转化后调用对应的方法
* 只接受function对象
* 返回Promise对象，以实现链式调用
* 定义：then(fulfilledHander, errorHandler, progressHandler)

###### 原理
将业务中不可变的部分封装在Deferred中，将可变部分交给了Promise

**图示**
![](https://gitee.com/cc12703/figurebed/raw/master/img/20201106225248.png)


#### 流程控制库

##### 尾触发与next
需要手工调用才能持续执行后续的调用




## 参考资料
* 《深入浅出Node.js》
* 《了不起的Node.js》