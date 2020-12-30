

# Odoo总体

## 概述
* 一个完整的中小企业ERP系统
* 本质上是一个快速实现各种定制化管理系统的框架工具

## 特点
* 高度模块化
* 开源免费
* 方便二次开发


## 版本
* Odoo 8.0 是最后一个全功能版本，之后分成社区版本和企业版本
* Odoo 10.0 稳定，三方模块也更全面
* Odoo 11.0 底层技术架构变化较大



## 架构

### 系统架构
#### 概述
* 数据库服务器: 使用PostgreSQL数据库
* 应用服务器： 包含所有业务逻辑代码
* 客户端：将数据库数据和业务数据渲染成html格式

#### 应用服务器
* orm：对象映射库，用于操作数据库
* bmd：基础模块
* report：生成各种报表（格式：pdf, html, OpenOffice）
* workflow：工作流引擎
* webService：提供网络调用接口（格式：xml-rpc, json-rpc）

#### 图示
![](https://gitee.com/cc12703/figurebed/raw/master/img/20210104145638.png)


### 应用架构
* 遵循MVC设计架构（Models，Views, Actions）
* Models：数据层，直接与数据库交互
* Actions：逻辑层，负责与数据层交互
* Views：视图层，负责展示数据，与用户交互
