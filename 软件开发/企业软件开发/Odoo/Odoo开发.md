

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
    /controllers/ -- 控制器（HTTP路由）
    /models/ -- 模型定义
    /views/ -- 视图和模板
    /demo/  -- 演示数据
    /security/ -- 安全配置
    /report/  -- 报表和模型
    /\_\_manifest\_\_.py -- 模块元数据


### manifest配置
* name 模块名
* summary 模块简介
* description 模块描述
* website 模块相关网址
* version 版本号
* depends 需要依赖的模块，默认为base模块
* data 视图文件

### base模块
#### 概述
* 用于提供重要的特性

#### 信息资源库
* 外部ID以 ir. 开头
* 用于存储一些基础功能

##### 例子
* ir.ui.menu 设置菜单
* ir.ui.view 用于视图
* ir.model  用于模型
* ir.model.fields 用于模型的字段

#### 资源
* 外部ID以 res. 开头
* 包括了与国际化相关的模型

##### 例子
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
#### 常规字段
* 字符串：fields.Char, fields.Text
* 整型：fields.Integer
* 布尔型：fields.Boolean
* 浮点型：fields.Float 
* 日期型：fields.Date, fields.Datetime
* 下拉列表框：fields.Selection(items, string)
    * items 选项列表
    * string 前端显示名

#### 字段属性
* string 前端界面显示的字段名称
* default 默认值
* required true表示不能为空
* help 前端界面显示的提示信息
* index true表示会创建索引
* readonly true表示前端界面不可编辑
* groups 限定可访问的安全组
* states 通过字典来设置界面相关的属性

#### 保留字段
* id 记录的唯一标识
* create_date 记录的创建日期
* write_date 记录的最后修改日期
* active false表示前端界面不会显示
* state 生命周期状态，Selection类型
* sequence 定义记录的顺序，可以在前端界面修改

#### 特殊字段
##### 引用字段
* 支持动态的关联关系
* 同一个字段可以引用多个模型

```python
discovers = fields.Reference(
    [('res.user', '用户'), ('res.partner', '合作伙伴')], 'bug发现者'
)
```

##### 计算字段
* 可以根据函数自动计算出值
* 使用了computes属性来指定函数


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


### 模型关系
#### 关系字段
* fields.Many2one  多对一
* fields.Many2many 多对多
* fields.One2Many  一对多

#### many2one
* 数据库会生成一个外键字段

##### 参数
* comodel 关联模型的名字
* string  描述信息
* ondelete 删除关联记录时该字段的行为
    * 默认，设置为空
    * restricted 报错，阻止关联记录被删除
    * cascade  一起被删除

#### many2many
* odoo会增加一个关系表来维护关系，默认表名：表名1_表名2_rel

##### 参数
* comodel 关联模型的名字
* relation 关联表的名字
* column1 本模型的关联字段名字
* column2 关联模型的关联字段名字
* string 描述信息


#### one2many
* many2one的逆向关系
* 可以让关联模型获取数据更方便
* 数据依然存在多(many)一方的底层表中

##### 参数
* comodel 关联模型的名字
* inverse_name 关联字段的名字


### 模型继承
#### 经典继承
* 最常用的继承方式
* 若_name不赋值，则不会创建新的模型

#### 原型继承
* _inherit字段赋值一个模型列表
* 若_name赋值，则会创建一个新的模型，会继承父模型的所有特征

#### 委托继承
* _inherits字段赋值一个模型列表
* 特点：记录会存储在两个表中
    * 父模型字段存储在父模型对于的表中
    * 新模型的新字段存储在新模型对于的表中
    * 通过主外键进行关联
* 优点：避免在多个表中重复存储数据

#### 传统的继承
* 允许子类修改父类定义的方法、字段
* 允许子类新增加字段、方法
* 若_name与父类一样，则保存在相同的数据表中

#### 委派
* 将子类的字段与父类的字段进行关联




### 层级结构
* 用于处理同一个模型的数据记录之间的关系





## ORM接口

### 修饰器
#### @api.multi 
* 处理记录集
* 使用self参数获取记录集

#### @api.model
* 在类上增加方法
* 使用self参数获取对象引用

#### @api.depends
* 用于触发计算字段的计算操作
* 格式：（字段1，字段2，...）

#### @api.constraints
* 用于评估和检查字段
* 在字段值变化时，会触发检查
* 格式：（字段1，字段2，...）

#### @api.onchange 
* 用于用户交互时自动更新相关联字段
* 在字段值变化时，会触发操作
* 操作变换后的值只会保存在内存中，不会更新到数据库


### 内置方法
#### 写入操作
* <model>.create(vals) 创建新记录
* <recordset>.write(vals)  更新记录
* <recordset>.unlink()    删除记录

#### RPC网页端
* read(fields) 读取记录，只包含指定字段，记录是字典形式
* searc_read() 在结果记录集上执行查找操作

#### 导入导出
* load(fields, data) 从csv文件中导入数据
* export_data(fields) 导出数据到csv文件


### 其他接口
#### 获取服务器环境
* self.env 获取当前用户的运行环境
* env.uid 当前会话中用户的ID
* env.contenxt 当前会话的上下文信息
* env[model.name] 获取指定模型的引用
* env.sudo(user)  获取指定用户的环境信息
* env.ref(extID)  使用外部ID获取记录


### 操作记录集
#### 查询
* search()  使用domain表达式获取记录
* search_count() 获取指定条件的记录数量
* browse()

#### 日期时间
* Datetime.now() 返回当前datetime的字符串
* Date.today()   返回当前日期的字符串
* rom_string(val) 将字符串转换为date,datetime对象
* to_string(val)  将date,datetime对象转换为字符串


#### 记录集
* x in recordset 判断记录是否在记录集中
* recordset.ids   返回记录ID的列表
* recordset.ensure_one()  检查是否为单例记录
* recordset.filtered(func) 过滤记录集
* recordset.mapped(func)   映射记录集，返回一个列表
* recordset.sorted(func)   排序记录集


### context
* 用于存储会话数据，按字典格式存放数据
* 用于前端ORM和后端ORM中

#### 用途
* 前端：将信息从一个视图传递到下一个视图
* 后端：提供本地设置和信号信息

### domain
* 用于筛选数据记录
* ORM会将该表达式转换为SQL查询语句

#### 表达式语法
* 是一个条件列表
* 每个条件是一个元组：('<field-name>', '<operator>', '<value>')
* 多个条件默认是 “与” 的关系，要满足所有条件

##### operator值
* 比较符号：<, >, >=, <=, !=,
* '=like'：匹配模式
    * ‘_’ 匹配任意一位字符
    * ‘%’ 匹配任意字符串
* ‘like’或‘ilike'：匹配%value%模式，大小写不敏感
* ’in'或'not in'：检测是否包含、不包含在列表中

##### 逻辑运算符
* 可选： & 与，| 或，! 非
* 格式：先写逻辑运算符，会对后面的两个元组起作用

##### 例子
```python
[('is_done', '=', False)]
['|', ('follower_ids', 'in', [user.partener_id.id]),
    '|', 
        ('user_id', '=', user.id),
        ('user_id', '=', False)
]
```



## 前端网页

### 概述
* 基于内嵌的CMS（内容管理系统）开发
* 网页由控制器进行转发
* 使用QWeb模板来制作页面

#### 例子
控制器
```python
class Main(http.Controller):
    @http.route('/hello',auth='public')
    def hello(self,**kwargs):
        return request.render('bug-website.hello')
```

模板 bug-website.xml
```xml
<odoo>
    <data>
      <template id="hello" name="Hello Template">
        <h1>Hello World !</h1>
      </template>
    </data>
</odoo>
```

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

### 路由映射
* 使用@http.route()进行

#### 参数
* route 请求的http路径
* type 请求类型，http, json
* auth 认证方法
    * user 已登录的用户
    * public 直接访问，无限制
    * none 总是可用，方法无法访问数据库库
* methods 请求的http方法，默认所有方法

#### url占位符
* /hello/<name> 框架会把url中name参数的值提取出去传给方法
* /hello/<int:user_id> 框架把url中user_id的值转换成int型传给方法
* /hello/<model("res.users"):user> 框架把url中user的值转换为res.users模型的记录对象传给方法

### 请求对象 
* http.request对象

#### 属性
* context 上下文信息，键值映射
* env     当前请求的环境
* session 当前请求的会话信息


## 后端视图

### 概述
* 视图通过xml文件定义
* 文件放置于views目录下
* 所有视图信息都会保存在数据库中

#### 通用定义结构
* 使用 <record> 元素，定义数据库表中的一条记录
    * name 指定要存储的表
    * id 记录的唯一标识

#### 通用属性
* 使用 <field> 元素定义
* name 视图名字
* model 关联的模型名字
* priority 视图优先级，值小的被先返回
* arch 视图布局
* groups_id 可查看、使用视图的用户组
* inherit_id 父视图

#### 类型
* 菜单视图：用于把 数据模型 -- 菜单 -- 视图 连接起来
* 表单视图：用于创建、编辑数据模型
* 列表视图：用于展示数据模型
* 搜索视图：用于对数据模型进行搜索和过滤
* 图形视图：以图表形式呈现模型中的数据


### 继承
* 使用inherit_id引用父视图
* 在arch标签内进行扩展
* 扩展方式：定义父视图中的元素，再引入修改

#### 定位
* 使用xpath在父视图的xml中进行定位
* 格式：expr = "//标签名[@属性]=‘属性值’"

```xml
<xpath expr="//field[@name]='is_done'" position="before">
    ...扩展内容
</xpath> 
```

##### position属性
* after 扩展内容添加在匹配节点之后
* before 扩展内容添加在匹配节点之前
* inside 扩展内容添加在匹配节点内部（默认值）
* repalce 扩展内容替换掉匹配节点

#### 例子

```xml
<record model="ir.ui.view" id="bug-manage.follower_form">
    <field name="name">follower</field>
    <field name="model">res.partner</field>
    <field name="inherit_id" ref="base.view_partner_form"/>
    <field name="arch" type="xml">
    <!-- 定位元素，简化形式 -->
    <field name="mobile" position="after">
        <field name="bug_ids"/>
    </field>
    </field>
</record>
```



### 菜单视图
* 使用 act_window 和 menuitem 实现

#### 菜单项
* 使用 <menuitem> 定义

##### 属性
* name 前端界面显示
* id 唯一标识
* parent 指定父菜单
* action 指定窗口动作

##### 例子
```xml
<menuitem name="bug管理系统" id="bug-manage.menu_root"/>

<menuitem name="bug管理" id="bug-manage.menu_1" parent="bug-manage.menu_root"/>

<menuitem name="bug列表" id="bug-manage.menu_1_list" parent="bug-manage.menu_1"
              action="bug-manage.action_window"/>
```

#### 窗口动作
* 使用 <act_window> 定义
* 告知界面要使用什么模型，提供哪些视图

##### 属性
* id 唯一标识，名字中最多只能有一个点
* res_model 要显示的模型
* view_mode 视图类型
* target 视图显示的位置
    * new 在对话框中显示
    * current 在主内容区域显示
* domain 域表达式，用于过滤记录

##### 例子
```xml
<act_window id="bug-manage.action_window"
    name = "bug-manage window"
    res_model = "bm.bug"
    view_model = "tree,form"
/>
```


### 表单视图
* 使用 <form> 定义
* 分成三部分：头部、主体、联系

##### 头部
* 包含文档的生命周期、步骤、相关的操作按键
* 生命周期表示：文档在当前所在的生命周期中的点
    * 使用State(状态)字段、Stage(阶段)字段

##### 主体
* 放置表单的数据元素
* 位于<sheet>节点内
* 组成区域
    * 标题、副标题（一般在左上角）
    * 按键框（一般在右上角）
    * 区域字段
    * 笔记本（一般在底部）



### 看板视图
* 使用 <kanban> 定义
* 主要使用html元素构成

#### 用途
* 用于展示简单的业务流程
* 将不同节点的工作放在不同的列上

#### 子元素
##### field
* 定义用来集合计算的字段、用在视图逻辑中的字段
    * name 字段名
    * sum, avg, min, max, count 计算方式，只能选一个

##### templates
* 定义一个QWeb模板列表
* 卡片可以分割成多个模板，至少要定义一个kanban-box标签
* 使用的是客户端QWeb




### 列表视图
* 使用 <tree> 定义

#### 属性
* default_order 默认排序，格式："field-name,field-name desc"

#### 子元素
* field 定义要显示的字段

```xml
<tree>
    <field name="name"/>
    <field name="is_closed"/>
    <field name="user_id"/>
</tree>
```


### 搜索视图
* 使用 <search> 定义
* 右上角会显示一个搜索框

#### 子元素
* field 定义要显示的字段
* filter 定义过滤条件

```xml
<search>
    <field name="name"/>
    <field name="is_closed"/>
    <field name="user_id"/>
</search>
```

### 图形视图
* 使用 <graph> 定义

#### 属性
* type 图形类型：bar, pie, line
* stacked true表示使用堆积形式


## 报表 

### 概述
* Odoo的默认报表引擎为 QWeb报表引擎
* 生成：报表模板 --> html文档 --> pdf文件
* 使用Python QWeb

### 创建
* 文件位于report目录下
* 使用report标签
* 数据会被写入 ir.actions.report.xml模型中

#### 文件格式
```xml
<odoo>
  <data>
    <report id="action_bug_report"
        string="bug"
        model="bm.bug"
        report_type="qweb-pdf"
        name="bug_stage.report_bug_template"
      />
      <!-- 模板内容 -->
  </data>
</odoo>
```

### 模板
* 使用QWeb模板格式
* 报表通常遵循基本框架

```xml
<template id="report_bug_template">
    <!-- 处理html的基本设置 -->
    <t t-call="web.html_container">
      <!-- 处理报表的页眉和页脚 -->
      <t t-call="web.external_layout">

         <div class="page">

           <!-- 报表头部内容 -->
           <t t-foreach="docs" t-as="o">
             <!-- 报表行内容 -->
           </t>
           <!-- 报表底部内容 -->

         </div>

      </t>
    </t>
</template>
```



## QWeb引擎

### 概述
* 一个基于XML的模板引擎
* 可生成HTML片段和页面
* 模板指令：xml标签内以‘t-’开头的属性
* 标签 \<t\> 不会被渲染

#### 表达式处理
* 客户端JavaScript：看板视图
* 服务器Python：用于报表、页面


### 模板
#### 创建
##### 格式1
* 定义在\<templates\>标签内
* 使用t-name 定义模板名
* 使用在看板视图中

```xml
<templates>
    <t t-name="temp-name">
        ...
    </t>
</templates>
```

##### 格式2
* 定义在\<template\>标签内
* 使用属性 id 定义模板名

```xml
<template id="template_id" name="template_name">
    ...
</template>
```

#### 继承
##### data类文件
* 定义在views目录下
* 使用xpath定位语句定位元素

```xml
<template id="唯一ID" inherit_id="父模板的ID">
    <xpath expr="定位语句"> 
        //扩展内容
    </xpath>
</template>>
```

##### 视图类文件
* 定义在static/src/xml目录下
* 使用jquery的选择器来定位

```xml
<template id="唯一ID" xml:space="preserve">
    <t t-extend="父模板的名字">
        <t t-jquery="定位语句">
            //扩展内容
        </t>
    </t> 
</template>>
```



### 通用指令
#### t-esc
* 输出表达式的值，可以自动过滤xss和html

```xml
<p><t t-esc="value"/></p>
```
当value为42时，输出
```xml
<p>42</p>
```

#### t-if
* 条件判断，属性值为true时，输出标签内容
* 表达式会被自动求值
* t-elif， t-else 添加条件分支

```xml
<div>
    <t t-if="condition">
        <p>ok</p>
    </t>
</div>

<div>
    <p t-if="user.birthday == today()">Happy bithday!</p>
    <p t-elif="user.login == 'root'">Welcome master!</p>
    <p t-else="">Welcome!</p>
</div>
```

#### t-foreach
* 循环处理
* t-as 定义循环中的当前值

##### 辅助变量
* $as_index 迭代索引值，从0开始
* $as_size 集合个数

```xml
<p t-foreach="[1, 2, 3]" t-as="i">
    <t t-esc="i"/>
</p>
或
<t t-foreach="[1, 2, 3]" t-as="i">
    <p><t t-esc="i"/></p>
</t>
```
输出
```html
<p>1</p>
<p>2</p>
<p>3</p>
```

#### t-set
* 创建变量
* 有t-value 表达式解析后，设置给变量
* 无t-value 节点的内容会被渲染，设置给变量

```xml
<t t-set="foo" t-value="2 + 1"/>

<t t-set="foo">
    <li>ok</li>
</t>
```

#### t-attr
* 创建属性
* 多种形式

##### t-att-$name
* 创建名为$name的属性
* 原属性值会被解析成新属性值
* 原属性值可以是一个表达式

```xml
<div t-att-a="42"/>  
```
输出
```html
<div a="42"></div>
```

##### t-attf-$name
* 创建名为$name的属性
* 原属性值是一个格式化字符串
* 格式：{{表达式}} 或 #{表达式}

```xml
<t t-foreach="[1, 2, 3]" t-as="item">
    <li t-attf-class="row {{ item_parity }}"><t t-esc="item"/></li>
</t>
```
输出
```html
<li class="row even">1</li>
<li class="row odd">2</li>
<li class="row even">3</li>
```

##### t-att=mapping
* 原属性值是映射表，每个键值对生成一个属性

```xml
<div t-att="{'a': 1, 'b': 2}"/>
```
输出
```html
<div a="1" b="2"></div>
```

##### t-att=pair
* 原属性值是一个元组

```xml
<div t-att="['a', 'b']"/>
```
输出
```html
<div a="b"></div>
```

#### t-call
* 调用其他模板
* 使用 t-set 传入参数

```xml
<t t-call="other-template-name">
    <t t-set="arg_max" t-vlaue="3" />
</t>
```



## 其他功能


### 动作Action
#### 概述
* 都继承自 ir.actions.actions模型

#### 窗口action
* 用于打开模型的各种视图
* 模型为ir.actions.act_window

##### 字段
* res_model 要显示的数据模型
* view_type 视图类型
* view_mode 视图类型列表，逗号分隔
* target 打开模式
    * current 当前窗口
    * new 新窗口
* context 额外的上下文信息
* domain 自动执行的记录过滤条件

#### 连接action
* 用于打开一个网站页面
* 模型为 ir.actions.act_url

##### 字段
* url 要打开的链接
* target 打开模式
    * new 新窗口打开（默认）
    * self 替换当前页面

#### 服务器action
* 用于触发服务端的动作
* 模型为 ir.actions.server

##### 字段
* model_id 要处理的模型
* code 要执行的python代码
* trigger 发送信息给信息流

##### 例子
```xml
<record model="ir.actions.server" id="记录id">
    <field name="name"></field>
    <field name="model_id" ref="模块名.model_下划线分隔的模型名"/>
    <field name="code">
        要执行的python代码。
    </field>
</record>
```

#### 客户端action
* 用于触发客户端的动作
* 模型为 ir.actions.client
* 通过core.action_registry.add()主动注册

##### 步骤
1. 在js中定义客户端的widget，并注册
1. 在视图中调用

##### 例子
```js
var customeWidgetName = Widget.extend({
        init: ... //init函数
        start: ... //自动调用到start函数

        //其他函数
})

core.action_registry.add('widget名', customeWidgetName);

return {
    widget名: widget名,
};
```

```xml
<record id="action_" model="ir.actions.client">
            <field name="name"></field>
            <field name="res_model"></field>
            <field name="tag">widget注册时的tag名</field>
</record>
```

#### 报表渲染action
* 用于报表渲染前的信息设置
* 模型为 ir.actions.report.xml

##### 字段
* name 名字
* model  数据的来源模型
* report_type 渲染格式，qweb-pdf, qweb-html
* report_name 报表名字



### 定时任务
* 用于定时完成一些操作
* 在夜间执行计算量大的操作
* 模型为 ir.cron 

#### 字段
* name 任务名字
* user_id 执行任务的用户，一般为base.user_root
* interval_number 任务执行的频次
* interval_type 执行频次的单位
    * 包含：minutes, hours, days, months, weeks
* numbercall 循环执行的次数，-1无限制
* model 任务方法所在的模块
* function 任务方法

#### 例子
```xml
<record id="ir_cron_scheduler_demo_action" model="ir.cron">
    <field name="name">Demo scheduler</field>
    <field name="user_id" ref="base.user_root"/>
    <field name="interval_number">2</field>
    <field name="interval_type">minutes</field>
    <field name="numbercall">-1</field>

    <field eval="False" name="doall"/>
    <field eval="'scheduler.demo'" name="model"/>
    <field eval="'process_demo_scheduler_queue'" name="function"/>
</record>
```


### 向导
#### 功能
* 类似于弹窗，用于接收用户输入，然后进行相应处理
* 用于用户输入复杂的参数

#### 模型
* 基类为models.TransientModel
* 不能使用one2Many关系（导致模型无法被清理）
* 记录不是永久的，一段时间后会被自动删除
* 不需要访问权限，用户有向导记录的所有权限

#### 视图
* 一般是form视图
* <footer> 用于放置动作按键



### 安全性
#### 访问控制
##### 概述
* 在security目录下创建ir.model.access.csv文件
* 系统启动时会将数据写入ir.model.access模型中

##### 字段
* id 记录的唯一标识
* name 记录名字，有唯一性，格式：<模块名>.<安全组名>
* model_id 设置访问权限的模块ID
* group_id 要授予权限的安全组的ID
* perm 对应的权限（read, write, create, unlink）