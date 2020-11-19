[TOC]

# webpack

## 总体

### 功能
* JavaScript应用的静态模块打包工具
* 为应用递归构建出一个模块的依赖关系图
* 将所有这些模块打包成一个、多个bundle

### 优点
* 支持多种模块标准：AMD, CommonJS, ES6
* 有完备的代码分割解决方案
* 可以处理各种类型的资源
* 有庞大的社区支持

### 关键概念

#### 入口(entry)
* 打包的起点
* 指定哪个模块作为构建依赖图的开始

#### 输出(output)
* 指定如何构建输出的bundles

#### 预处理器(loader) 
* 用于处理非JavaScript文件
* 将所有类型的文件转换成有效模块

#### 插件(plugin)
* 用于解决loader无法实现的事情

#### 模式(mode)
* 用于启用对应的优化功能
* 选项：development, production



## 原理

### 打包原理
* 给每个模块创建了一个可以导出、导入的环境
* 不会修改代码的执行逻辑


### bundle内容
* 最外层为立即可执行匿名函数，构成局部的作用域
* installedModules对象，储存模块加载执行后的值
* \_\_webpack_require\_\_ 函数，模块加载的具体实现
* modules对象，存储所有产生了依赖关系的模块


### bundle运行
1. 外层匿名函数会初始化执行环境
2. 加载入口模块
3. 执行模块代码
    1. 遇到module.exports时会记录下导出值
    2. 遇到require函数，进入\_\_webpack_require\_\_函数内部运行
4. \_\_webpack_require\_\会判断模块是否在installedModules中
    1. 若已在，则直接取值
    2. 若不在，则进入步骤3
5. 所有依赖模块都加载完成后，继续执行入口模块






## 配置


### 配置文件
* 文件名：webpack.config.js
* 内容：通过module.exports导出一个对象

### 配置封装

#### 使用相同配置文件
* 生产环境、开发环境都使用webpack.confg.js
* 给环境变量ENV设置不同的值
* 在配置文件中判断该环境变量

#### 使用不同配置文件
* 每个环境单独一个文件 webpack.xxxx.config.js
* 通过--config传入不同文件
* 使用webpack.common.config.js放置公共配置



### 入口

#### 配置项
* context：入口的基础目录，必须使用绝对路径
* entry：入口文件，可以使用字符串、数组、对象、函数
* vendor：需要单独打包的第三方模块

#### 例子
```js
const path = require('path')
//单页应用
module.exports = {
    context: path.join(__dirnamne, './src')
    entry: {
        app: './src/app.ja'
        vender: ['react'] //需要单独打包的第三方模块
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

### 出口(output)

#### 配置项
* filename：输出bundle的名字，可以是字符串、模板变量
* path：输出目录，必须是绝对路径
* publicpath：资源的请求位置，指JS,CSS所请求的间接资源路径

#### 模板变量
* [hash] ： 打包所有资源生成的hash值
* [chunkhash] ： 当前chunk内容的hash
* [id]：当前chunk的ID
* [name]：当前chunk的名字

#### 例子
```js
const path = require('path')
module.exports = {
    output: {
        filename: 'bundle.js',
        path: path.join(__dirname, 'dist'),
        publicPath: "/assets/",
    } 
}


module.exports = {
    output: {
        filename: '[name]@[chunkhash].js'  //可读性高，可以控制客户端缓存
    } 
}
```

### 构建目标

#### 配置项
* target: 指定一个环境
    * electron-main 编译为Electron主进程可用
    * electron-renderer 编译为Electron渲染进程可用
    * node 编译为Node.js环境可用
    * web 编译为浏览器环境可用（默认）

#### 例子
```js
module.exports = {
  target: 'node'
};
```

### 模块(module)

* 用于配置如何处理不同类型的模块
* 由一个规则数组组成
* 每个规则包括：条件、结果、嵌套规则
* 条件有两种：resource, issuer
    resource: 请求资源
    issuer: 被请求资源，导入的地方
* 规则结果只有在条件匹配上时才会使用

#### 条件说明
> 从 app.js 导入 ./style.css
> issuer 就是 /path/to/app.js
> resource 就是 /path/to/style.css


#### 配置项
* rule.issuer 配置发布者，引用资源者
* rule.resource 配置资源

* rule.xxx.test 规则匹配条件，使用正则表达式
* rule.xxx.include 规则生效条件
* rule.xxx.exclude 排除特定条件，优先级比include高

* rule.use: 配置规则所使用的loader，一个或多个
* rule.enforce 指定执行顺序
    * normal：正常执行，直接定义的loader都属于该类型
    * pre：预执行，在所有正常loader之前执行
    * post: 后执行，在所有正常loader之后执行

* rule.test: 等于 rule.resource.test
* rule.include 等于 rule.resource.include
* rule.exclude 等于 rule.resource.exclude


#### 例子
```js

rules:[{
    test: /\.css/,
    use: ['style-loader', 'css-loader'],
    exclude: /node_modules/,
    include: /src/,
}]


rules:[{ 
    test: /\.js$/,
    enforce: 'pre',
    use: 'eslint-loader'
}]
```

### 解析(resolve)

* 用于配置模块如何被解析

#### 配置项

* alias: 设置路径别名
* extensions: 设置自动解析的文件后缀名，默认为 .js, .json 

#### 例子
```js
alias: {
  Utilities: path.resolve(__dirname, 'src/utilities/'),
  Templates: path.resolve(__dirname, 'src/templates/')
}
```

### 插件(plugins)

* 用于配置需要使用的插件列表

#### 例子
```js
plugins: [
  // 构建优化插件
  new webpack.optimize.CommonsChunkPlugin({
    name: 'vendor',
    filename: 'vendor-[hash].min.js',
  }),
  new webpack.optimize.UglifyJsPlugin({
    compress: {
      warnings: false,
      drop_console: false,
    }
  })
```


## 预处理器


### 概述

* 本质上是一个函数，格式：output = loader(input)
* 可以进行链式操作，对一种资源设置多个loader
* 多个loader时，webpack按从后往前的顺序调用


### babel-loader
* 用于处理ES6+语法，并将其编译成ES5

#### 子模块
* babel-loader：使babel与webpack协同工作
* @babel/core：Babel编译器
* @babel/preset-env：官方的预置器，用于自动添加插件和补丁

#### 配置
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

#### 其他
* 支持从.babelrc中读取配置



### ts-loader
* 用于连接webpack和typescript模块

#### 配置
```js
{
    test: /\.ts$/
    use: 'ts-loader'
}
```

#### 其他
* typescript的配置在tsconifg.json中



### html-loader
* 用于将一个html片段通过js加载进行
* 将html文件转化为字符串并进行格式化

#### 配置
```js
{ 
    test: /\.handlebars$/,
    use: 'handlebars-loader'
}
```

#### 使用
* 加载文件，得到一个函数
* 调用函数获得最终的字符串

```js
import template from './content.handlebars'

html = template({
    title: "Title"
})

```


### file-loader

#### 功能
* 用于打包文件类型的资源(图片)
* 可以配置输出路径 output.publicPath
* 默认文件名为文件的hash值

#### 配置
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

### vue-loader

#### 功能
* 用于处理vue组件

#### 子模块
* vue-loader： 拆分组件的模板，js，样式
* vue：vue框架
* vue-template-compiler：编译vue模板
* css-loader：处理样式

#### 配置
```js
{
    test: /\.vue$/,
    use: 'vue-loader'
}
```

### css-loader 

#### 功能
* 处理CSS的加载语法
* 启用CSS模块化
    * 每个CSS文件有单独的作用域
    * 对CSS进行依赖管理
    * 通过composes复用其他CSS

#### 配置项
* modules: 是否启用模块化
* localIdentName: 指定如何编译CSS中的类名
    * name 替换成CSS文件名，
    * local 替换成选择器标识符
    * hash 替换成5位的hash值，防止冲

#### 例子
```js
{
    loader: 'css-loader',
    options: {
        modules: true, 
        localIdentName: '[name]__[local]__[hash:base64:5]', 
    }
}
```


### sass-loader

#### 功能
* 处理sass语法
* 需要安装node-sass

#### 例子
```js
{
    test: /\.scss$/,
    use: ['style-loader','css-loader', 'sass-loader']
}
```

### postcss-loader

#### 功能
* 一个编译插件的容器
* 接收样式代码，交由编译插件处理，输出CSS
* 需要使用postcss.config.js进行配置

#### 例子
```js
{
    test: /\.css/,
    use: ['style-loader', 'css-loader', 'postcss-loader]
}
```

#### CSS功能
* 自动前缀，使用Autoprefixer，自动添加厂商前缀
* 质量检测，使用stylelint



## 插件

### DefinePlugin

#### 功能
* 配置全局变量

#### 例子
```js
new webpack.DefinePlugin({
    ENV: JSON.stringify('production'),
    ENV_ID: 1333333,
    IS_PRODUCTION: true
})
```

### EnvironmentPlugin

#### 功能
* 快捷设置process.env的环境变量
* 最终也会使用 DefinePlugin

#### 例子
```js
new webpack.EnvironmentPlugin(['NODE_ENV', 'DEBUG'])


new webpack.EnvironmentPlugin({
  NODE_ENV: 'development', // 除非有定义 process.env.NODE_ENV，否则就使用 'development'
  DEBUG: false
})
```



## 其他

### dev-server
* 调用webpack进行模块打包，并处理结果的资源请求
* 作为普通的web-server，处理静态资源文件请求
* 支持live-reloading功能
* 支持hot-module-replacement功能


### 缓存

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




## 参考资料

* https://www.webpackjs.com/concepts/