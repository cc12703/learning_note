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
const path = require('path')
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
        pageA: './src/pageA.js'
        pageB: './src/pageB.js'
        vender: ['react']
   }

}
```

#### 出口
* filename：输出资源文件名，可以是字符串、模板变量
* path：资源输出位置，必须是绝对路径
* publicpath：资源的请求位置，指JS,CSS所请求的间接资源路径


##### 文件名动态配置
* [hash] ： 打包所有资源生成的hash值
* [chunkhash] ： 当前chunk内容的hash
* [id]：当前chunk的ID
* [name]：当前chunk的名字

##### 例子
```js
const path = require('path')
module.exports = {
    output: {
        filename: 'bundle.js',
        filename: '[name]@[chunkhash].js'  //可读性高，可以控制客户端缓存
        path: path.join(__dirname, 'dist')
    } 
}
```






## 预处理器

### 一切皆模块
* 所有的静态资源都是模块，可以像加载js文件一样处理

### loader

#### 概述
* 本质上是一个函数，格式：output = loader(input)
* 可以进行链式操作，对一种资源设置多个loader
* 多个loader时，webpack按从后往前的顺序调用

#### 引入
* test: 接收一个正则表达式，只有匹配上的文件才会使用该条规则
* use: 接收一个数组，包含该规则所使用的loader
* exclude: 接收一个正则表达式，排除所有被正则匹配到的模块，优先级更高
* include: 接收一个正则表达式, 规则只对正则匹配到的模块生效

```js
module.exports = {
    //...
    module: {
        rules:[{
            test: /\.css/,
            use: ['style-loader', 'css-loader'],
            exclude: /node_modules/,
            include: /src/,
        }]
    }
}
```

#### 执行顺序
* normal：正常执行，直接定义的loader都属于该类型
* pre：预执行，在所有正常loader之前执行
* post: 后执行，在所有正常loader之后执行

```js
rules: [
    { 
        test: /\.js$/,
        enforce: 'pre',
        use: 'eslint-loader'
    }
]
```



### 各种loader

#### babel-loader
* 用于处理ES6+语法，并将其编译成ES5

##### 子模块
* babel-loader：使babel与webpack协同工作
* @babel/core：Babel编译器
* @babel/preset-env：官方的预置器，用于自动添加插件和补丁

##### 配置
```js
{
    test: /\.js$/
    exclude: /node_modules/, //排除该目录
    use: {
        options: {
            cacheDirectory: true, //启动缓存机制，防止重复打包
            presets: [[
                'env', { 
                    modules: false //阻止babel将ES6 Module转化为CommonJS形式
                }
            ]]
        }
    } 
}
```

##### 其他
* 支持从.babelrc中读取配置

#### ts-loader
* 用于连接webpack和typescript模块

##### 配置
```js
{
    test: /\.ts$/
    use: 'ts-loader'
}
```

##### 其他
* typescript的配置在tsconifg.json中



#### html-loader
* 用于将一个html片段通过js加载进行
* 将html文件转化为字符串并进行格式化

##### 配置
```js
{ 
    test: /\.handlebars$/,
    use: 'handlebars-loader'
}
```

##### 使用
* 加载文件，得到一个函数
* 调用函数获得最终的字符串

```js
import template from './content.handlebars'

html = template({
    title: "Title"
})

```


#### file-loader
* 用于打包文件类型的资源(图片)
* 可以配置输出路径 output.publicPath
* 默认文件名为文件的hash值

##### 配置
```js
{
    test: /\.(png|jpg|gif)$/,
    use: {
        loader: "file-loader",
        options: {
            name: '[name].[ext]',
            publicPath: './another-path'
        }
    }
}
```

#### vue-loader
* 用于处理vue组件

##### 子模块
* vue-loader： 拆分组件的模板，js，样式
* vue：vue框架
* vue-template-compiler：编译vue模板
* css-loader：处理样式

##### 配置
```js
{
    test: /\.vue$/,
    use: 'vue-loader'
}
```


## 样式处理

* css-loader：处理CSS的加载语法
* style-loader：把样式插入页面


### Sass

* sass语法是对css语法的增强
* css的预处理器
* 需要安装sass-loader, node-sass

#### 配置
```js
{
    test: /\.scss$/,
    use: ['style-loader','css-loader', 'sass-loader']
}
```

### PostCSS
* 一个编译插件的容器
* 接收样式代码，交由编译插件处理，输出CSS
* 需要安装postcss-loader
* 需要使用postcss.config.js进行配置

#### 配置
```js
{
    test: /\.css/,
    use: ['style-loader', 'css-loader', 'postcss-loader]
}
```

#### 功能
* 自动前缀，使用Autoprefixer，自动添加厂商前缀
* 质量检测，使用stylelint


### CSS模块化

#### 特点
* 每个CSS文件有单独的作用域
* 对CSS进行依赖管理
* 通过composes复用其他CSS

#### 启用
* 开启css-loader中modules配置项
* localIdentName 指定如何编译CSS中的类名
    * name 替换成CSS文件名，
    * local 替换成选择器标识符
    * hash 替换成5位的hash值，防止冲突

```js
{
    loader: 'css-loader',
    options: {
        modules: true, 
        localIdentName: '[name]__[local]__[hash:base64:5]', 
    }
}
```

## 生产环境

### 配置封装

#### 使用相同配置文件
* 生产环境、开发环境都使用webpack.confg.js
* 给环境变量ENV设置不同的值
* 在配置文件中判断该环境变量

#### 使用不同配置文件
* 每个环境单独一个文件 webpack.xxxx.config.js
* 通过--config传入不同文件
* 使用webpack.common.config.js放置公共配置

### production模式
* webpack会自动添加生产环境的配置项

```js
//webpack.config.js
module.exports = {
    mode: 'production',
}
```

### 环境变量
* 使用DefinePlugin进行设置

```js
//webpack.config.js
const webpack = require('webpack')
module.exports = {
    plugins: {
        new webpack.DefinePlugin({
            ENV: JSON.stringify('production'),
            ENV_ID: 1333333,
            IS_PRODUCTION: true
        })
    }
}
```

## 缓存

### 问题
* 资源修改后，需要更改资源的URL迫使客户端下载新资源

### 资源hash
* 打包时将资源内容算一下hash值
* 将hash值作为版本号存放在文件名中

```js
module.exports = {
    output: {
        filename: 'bundle@[chunkhash].js'
    }
}
```

### 输出动态HTML
* 打包后自动将资源名同步到引用路径中
* 使用html-webpack-plugin
    * 自动生成一个index.html