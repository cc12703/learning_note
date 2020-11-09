[TOC]

# webpack

## 总体

* JavaScript模块打包工具
* 核心功能：解决模块之间的依赖
* 将各个模块组织在一起，最终合并为一个Js文件

### 优点
* 支持多种模块标准：AMD, CommonJS, ES6
* 有完备的代码分割解决方案
* 可以处理各种类型的资源
* 有庞大的社区支持

### 配置文件
* 文件名：webpack.config.js
* 内容：通过module.exports导出一个对象

### dev-server
* 调用webpack进行模块打包，并处理结果的资源请求
* 作为普通的web-server，处理静态资源文件请求
* 支持live-reloading功能
* 支持hot-module-replacement功能


## 模块

### 语法

#### CommonJS
* 最初为服务端设计
* 每个文件是一个模块，有一个局部的作用域
* 使用module.exports进行导出
* 使用module对象来存放当前模块的信息
* 使用require进行导入
* 导入模块时，模块只会被加载一次并存入缓存

##### 特点
* 模块依赖关系的建立是动态的，发生在代码运行阶段
* 导入模块时，获取的是一份导出值的拷贝

```js
module.exports = {
    name: 'Calculater',
    add: function(a, b) {
        return a + b;
    }
}

const calculator = require('./calculator.js')
```


#### ES6
* 每个文件是一个模块，有一个局部的作用域
* 使用export进行命名导出，可以有多个
* 使用export default进行默认导出，只能有一个
* 使用import {} 来导入命名导出
* 使用import * as 来整体导入
* 使用import 来导入默认导出
* 使用as来进行导入、导出的重命名

##### 特点
* 模块依赖关系的建立是静态的，发生在代码编译阶段
* 导入模块时，获取的是一个动态映射，且不可改变


##### 静态优点
* 可以进行死代码的检测和排除
* 可以进行模块变量类型检查
* 可以进行编译器优化

```js
export const name = 'calculator'
export add = function(a, b) { return a + b; }

export default {
    name: 'calculator'
    add: function(a, b) {
        return a + b;
    }
}


import { name, add as getSum } from './calculator.js'
import * as calculator from './calculator.js'
import React, { Component } from 'react'

```


### npm实现

#### 安装
* npm install xxx --save
* 模块被安装在工程的node_modules目录下
* 依赖信息记录在package.json中

#### 加载
* import xxx from 'lodash'  加载整个模块
* import xxx from 'lodash/fp/all.js' 加载模块内部的某个文件

#### 定义
* 每个模块都有一个入口
* 入口定义在模块package.json的main字段中


### 打包原理
* 给每个模块创建了一个可以导出、导入的环境
* 不会修改代码的执行逻辑


#### bundle内容
* 最外层为立即可执行匿名函数，构成局部的作用域
* installedModules对象，储存模块加载执行后的值
* \_\_webpack_require\_\_ 函数，模块加载的具体实现
* modules对象，存储所有产生了依赖关系的模块


#### bundle运行
1. 外层匿名函数会初始化执行环境
2. 加载入口模块
3. 执行模块代码
    1. 遇到module.exports时会记录下导出值
    2. 遇到require函数，进入\_\_webpack_require\_\_函数内部运行
4. \_\_webpack_require\_\会判断模块是否在installedModules中
    1. 若已在，则直接取值
    2. 若不在，则进入步骤3
5. 所有依赖模块都加载完成后，继续执行入口模块


## 资源

* entry：入口，打包开始的位置
* chunk：代码块，被抽象、包装过后的一些模块
* bundle：打包后的产物
* 一个entry对应生成一个bundle

### 流程
1. 从entry开始检索
2. 将有依赖关系的模块生成一棵依赖树
3. 最终得到chunk和bundle

### 配置

#### 入口
* context：资源入口的路径前缀，必须使用绝对路径
* entry：入口文件路径，可以使用字符串、数组、对象、函数
* vendor：需要单独打包的第三方模块

```js
//单页应用
module.exports = {
    context: path.join(__dirnamne, './src')
    entry: {
        app: './src/app.ja'
        vender: ['react']
   }
}


//多页应用
module.exports = {
    context: path.join(__dirnamne, './src')
    entry: {
        pageA: './src/pageA.ja'
        pageB: './src/pageB.js'
        vender: ['react']
   }

}
```

#### 出口
* fileName：输出资源文件名，可以是字符串、模板变量
* path：资源输出位置，必须是绝对路径
* publicPath：资源的请求位置，指JS,CSS所请求的间接资源路径


## 预处理器

### 一切皆模块
* 所有的静态资源都是模块，可以像加载js文件一样处理