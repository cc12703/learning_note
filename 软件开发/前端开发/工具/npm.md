[TOC]

# npm学习


## 总体

* 是javascript的包管理工具
* 随node.js一起发布的



## 模块化支持

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

## 命令

### 更新
```
npm install npm@latest -g
```

### 创建描述文件
* 描述文件为 package.json

```
npm init 
npm init --yes  //使用默认配置
```

### 安装包

#### 参数
* -D --save-dev 将包写入package.json的devDependencies中
* -S --save  将包写入package.json的dependencies中
* -E --save-exact 精确指定包的版本

```
npm install <package-name>  //安装模块到node_modules目录下
npm install               //读取pacakge.json来安装模块
npm install <package-name> --save   //安装后将模块信息记录到package.json中
npm install <package-name> --save-dev

npm install -g <package-name>  //全局安装模块
```

### 更新
```
npm update        //更新当前目录模块
npm update -g     //更新全局模块

```

### 卸载
```
npm uninstall <package-name>
npm uninstall -g <package-name>
```

### 配置源

#### 临时使用
```
npm install express --registry https://registry.npm.taobao.org
```

#### 全局使用
```
npm config set registry https://registry.npm.taobao.org
npm config get registry   //验证是否成功
```


## 配置

### 配置加载优先级
1. 命令行参数
2. 环境变量
3. 项目级别的rc文件 (project/.npmrc)
4. 用户级别的rc文件 (~/.npmrc)
5. 全局的rc文件 (/etc/npmrc)
6. 内置的rc文件 (npm/npmrc)


## 描述文件

### 字段说明

#### name 
* 模块名
* 要全部小写，无空格，可以用下划线和破折号

#### version 
* 版本信息

#### main  
* 主入口文件
* 模块引入后，首先被加载的文件
* 默认为index.js

#### bin
* 指定可以执行文件
* 格式为KV对，cmd-name : path

#### scripts 
* 定义常用命令
* 格式为KV对，event-name : command

#### XXXdependencies
* 定义依赖模块列表
* 格式为KV对，mod-name : mod—version

##### 版本格式
* version 精确版本
* > version 大于某版本
* < version 小于某版本
* ~ version 约等于某版本
* ver1 - ver2 等于 ver1 <= xxx <= ver2

##### 类型
* dependencies 正常依赖
* devDependencies 开发额外依赖
* bundledDependencies  发布时被一起打包的模块
* peerDependencies 作为插件的依赖（需要在宿主工程上安装）
* optionalDependencies 可选依赖，安装失败时npm会继续运行

#### config 
* 存放不易变化的配置信息

#### private 
* 设置为true时，私有模块将无法发布


### 依赖版本格式
* 1.2.2   只安装指定版本
* ~1.2.2  安装1.2.x的最新版本，要大于等于1.2.2
* ^1.2.2  安装1.x.x的最新版本，要大于等于1.2.2
* latest  安装最新版本