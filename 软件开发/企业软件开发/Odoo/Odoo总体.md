

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


## 配置

* 配置文件为 odoo.conf中

### 配置项
* addons_path 插件模块查找路径，可以多个，逗号分隔
* data_dir 数据目录，用于存放session数据、附件、缓存文件

* db_host 数据库主机名
* db_port 数据库端口号
* db_user 数据库用户名
* db_password 数据库密码
* db_maxconn 数据库最大连接数
* db_name 要预加载的数据库，可以多个，逗号分隔

* log_level 日志级别，可选值：debug,debug_sql,info,warn,error,critical
* logfile 存储日志的文件
* logrotate 是否按天存放日志
* log_handler 模块的日志级别，格式：module:log_level，默认：:INFO(所有模块都是info级别)

* workers 进程数量，大于0则启用多进程模式，默认：0
* max_cron_threads 定时任务所使用的worker数量，默认：2


## 调试

### 调试模式
* 启动odoo时，使用命令行参数 --dev=feature,feature,feature

#### 特性
* xml 直接从xml文件中读取QWeb模板
* reload 当python文件修改后重启服务
* qweb 当节点包含t-debug='debugger'时，相关模板运行被中断
* all 激活所有特性


### 开发者模式

#### 激活
1. 以admin登录
1. 进入 Settings 菜单
1. 找到 Share the love 板块
1. 点击对应链接

#### 其他方式激活
* 通过URL中插入debug
* chrome通过Odoo Debug插件

#### 功能
* 鼠标在表单视图字段上、列表视图列名上时，会出现提示信息，提供字段技术信息
* 会出现调试按键
* dev mode with assets：发送给浏览器的代码没有做最小化处理，用于调试js代码


### VSCode配置

#### 项目settings.json
```json
{
    // 引用odoo源代码，用于自动完成，转到定义
    "python.autoComplete.extraPaths": [
        "odoo-source-path"
    ],
    "python.analysis.extraPaths": [
        "odoo-source-path"
    ] 
}
```
