[TOC]

# npm学习


## 总体

* 是javascript的包管理工具
* 随node.js一起发布的


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
* name 模块名，要全部小写，无空格，可以用下划线和破折号
* version 版本信息
* main  模块引入后，首先被加载的文件。默认为index.js
* scripts 定义常用命令
* dependencies 依赖包列表
* devDependencies 开发额外依赖包列表
* bundledDependencies  发布时被一起打包的模块
* config 存放不易变化的配置信息
* private 设置为true时，私有模块将无法发布


### 依赖版本格式
* 1.2.2   只安装指定版本
* ~1.2.2  安装1.2.x的最新版本，要大于等于1.2.2
* ^1.2.2  安装1.x.x的最新版本，要大于等于1.2.2
* latest  安装最新版本