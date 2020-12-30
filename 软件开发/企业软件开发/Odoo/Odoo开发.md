

# Odoo开发

## 概述

### 原则
* 不建议直接修改源码
* 最好是新建一个模块，继承已有的模块进行扩展增强

### 应用 vs 模块
* 应用是一个独立的系统，在Odoo里有一个顶级菜单
* 模块是一组较为独立的功能

### 继承机制
* 允许自定义模块扩展现有的模块
* 层次：数据模型、业务逻辑、用户界面

## 模块

### 文件结构
> /root/
    /controllers/
    /models/
    /views/ -- 视图配置
    /demo/  -- 演示数据
    /security/ -- 安全控制
    /\_\_manifest\_\_.py -- 模块元数据


### manifest配置
* name 模块名
* summary 模块简介
* description 模块描述
* website 模块相关网址
* version 版本号
* depends 需要依赖的模块，默认为base模块
* data 视图文件

## base模块

### 概述
* 用于提供重要的特性

### 信息资源库
* 外部ID以 ir. 开头
* 用于存储一些基础功能

#### 例子
* ir.ui.menu 设置菜单
* ir.ui.view 用于视图
* ir.model  用于模型
* ir.model.fields 用于模型的字段

### 资源
* 外部ID以 res. 开头
* 包括了与国际化相关的模型

#### 例子
* res.partner 联系人相关
* res.company 公司相关
* res.currency 货币相关



## 视图