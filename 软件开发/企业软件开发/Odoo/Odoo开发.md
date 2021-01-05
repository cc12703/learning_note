

# Odoo开发

[TOC]

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


## 模型

### 创建
* 文件放置于models目录下
* 需要继承models.Model类
* fields用于定义字段

```python
class Bug(models.Model):
    _name='bm.bug'
    _description='bug'

    name=fields.Char('bug简述',required=True)
    detail=fields.Text(size=150)
    is_closed=fields.Boolean('是否关闭')

    user_id=fields.Many2one('res.users',string='负责人')
    follower_id=fields.Many2many('res.partner',string='关注者')
```

#### 模型名规则
* 保证唯一性
* 以模块名为前缀


### 模型属性
* _name 类的唯一标识，其他类通过该字段引用本类
* _description 类似于标签，提高查询友好性
* _order 显示时默认的排序字段
* _inherit 使用传统继承机制的父类，多个父类
* _inherits 使用委派机制的父类，多个父类


### 字段
#### 字段属性
* string 前端界面显示的字段名称
* required true表示不能为空
* help 前端界面显示的提示信息
* index true表示会创建索引

#### 保留字段
* id 记录的唯一标识
* create_date 记录的创建日期
* write_date 记录的最后修改日期

### 类型
#### 持久模型
* 继承models.Model类
* 使用对应的数据库表来保存数据

#### 瞬态模型
* 继承models.TransientModel类
* 使用数据库表来保存数据
* 数据是临时存储，会定期删除老数据

#### 抽象模型
* 继承models.AbstractModel类
* 数据不会被存储
* 为了方便复用字段、方法


### 模型继承
#### 传统的继承
* 允许子类修改父类定义的方法、字段
* 允许子类新增加字段、方法
* 若_name与父类一样，则保存在相同的数据表中

#### 委派
* 将子类的字段与父类的字段进行关联



## 视图
### 概述
* 通过xml文件定义
* 框架会解析xml文件并生成html文件
* 文件放置于views目录下

### 类型
#### 菜单
* 使用menuitem进行定义
* name 前端界面显示
* id 唯一标识
* parent 指定父菜单
* action 指定窗口动作

```xml
<menuitem name="bug管理系统" id="bug-manage.menu_root"/>

<menuitem name="bug管理" id="bug-manage.menu_1" parent="bug-manage.menu_root"/>

<menuitem name="bug列表" id="bug-manage.menu_1_list" parent="bug-manage.menu_1"
              action="bug-manage.action_window"/>
```

#### 窗口动作
* 用途：将模型的数据显示出来
* model 固定为 ir.actions.act_window (数据保存在该表中)
* id 唯一标识，名字中最多只能有一个点
* res_model 要显示的模型
* view_mode 视图类型

```xml
<record model="ir.actions.act_window" id="bug-manage.action_window">
    <field name="name">bug-manage window</field>
    <field name="res_model">bm.bug</field>
    <field name="view_mode">tree,form</field>
</record>
```

#### 列表
* model 固定为 ir.ui.view
* arch 节点下使用 tree

```xml
<record model="ir.ui.view" id="bug-manage.list">
    <field name="name">bug列表</field>
    <field name="model">bm.bug</field>
    <field name="arch" type="xml">
    <tree>
        <field name="name"/>
        <field name="is_closed"/>
        <field name="user_id"/>
    </tree>
    </field>
</record>
```

#### 业务文档表单
* arch 节点下使用form
* 需要定义 header, sheet元素
    * header 显示在表单上方
    * sheet 视图的主题部分


#### 搜索
* arch 节点下使用 search

```xml
<record model="ir.ui.view" id="bug-manage.search">
    <field name="name">bug搜索</field>
    <field name="model">bm.bug</field>
    <field name="arch" type="xml">
    <search>
        <field name="name"/>
        <field name="is_closed"/>
        <field name="user_id"/>
    </search>
    </field>
</record>
```

### 继承
* 使用inherit_id引用父视图
* arch节点后指定要增加视图的锚点

```xml
<record model="ir.ui.view" id="bug-manage.follower_form">
    <field name="name">follower</field>
    <field name="model">res.partner</field>
    <field name="inherit_id" ref="base.view_partner_form"/>
    <field name="arch" type="xml">
    <!-- 扩展锚点 -->
    <field name="mobile" position="after">
        <field name="bug_ids"/>
    </field>
    </field>
</record>
```


## 业务逻辑

### 创建
* 可以使用模型类中的方法来实现
* 方法来用@api.multi来装饰


## 安全性

### 访问控制
#### 概述
* 在security目录下创建ir.model.access.csv文件
* 系统启动时会将数据写入ir.model.access模型中

#### 字段
* id 记录的唯一标识
* name 记录名字，有唯一性，格式：<模块名>.<安全组名>
* model_id 设置访问权限的模块ID
* group_id 要授予权限的安全组的ID
* perm 对应的权限（read, write, create, unlink）


## 网页

### 概述
* 网页由控制器进行转发
* 使用QWeb渲染引擎进行渲染

### 控制器
* 代码位于controllers目录下
* 类继承于http.Control类
* 使用@http.route指定路由路径

```python
class Bug(http.Controller):

    @http.route('/bug-manage')
    def BugManage(self, **kwargs):
        bugs=http.request.env['bm.bug']
        domain_bug=[('is_closed','=',False)]
        bugs_open=bugs.search(domain_bug)
        return http.request.render('bug-manage.bugs_template',{'bugs_open':bugs_open})
```