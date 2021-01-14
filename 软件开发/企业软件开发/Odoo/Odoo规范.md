


# Odoo规范

[TOC]




## 模块开发

### 文件存放
> /root/
    controllers/ -- 路由控制器（HTTP路由）
    models/ -- 模型定义
    views/ -- 视图和模板
    demo/  -- 演示数据
    security/ -- 安全配置
    report/  -- 报表和模型
    static/  -- 网页资源
    wizard/  -- 临时模型、及其视图
    \_\_manifest\_\_.py -- 模块元数据


### 文件命名
#### security
* ir.model.access.csv  定义模型访问权限
* \<module\>_groups.xml 定义安全组
* \<module\>_security.xml 定义访问规则

#### views
* \<model\>_views.xml  定义后台视图
* xxx_menus.xml 定义菜单项
* xxx_templates.xml 定义QWeb模板
* assets.xml  定义资源包（导入js和css）


### XML文件
* id属性放在model属性前面
* field标签中，先定义name属性

#### XML ID
##### 菜单项
* \<model\>_menu 
* \<model\>_menu_xxx 子菜单项
* \<model\>_menu_root 根菜单项

##### 视图
* \<model\>_view_\<type\>
    * type包括 kanban, form, tree, search

##### 动作
* \<model\>_action 主动作
* \<model\>_actions_\<detail\> 子动作

##### 窗口动作
* \<model\>_action_view_\<view-type\>

##### 安全组
* \<model\>_group_\<name\>\

##### 访问规则
* \<model\>_rule_\<name\>


### 编程命名

#### 模型名
* 使用单数形式 \<model\>.\<name\>
* 临时模型：\<related_base_model\>.\<action\>
    * related_base_model为临时模型关联的基模型名字
* report模型：\<related_base_model\>.report.\<action\>

#### 类名
* 使用驼峰命令

```python
class AccountInvoice(models.Model):
    ...
```

#### 变量名
* 模型变量（指向模型本身）使用驼峰命令
* 普通变量使用下划线小写字母
* 包含记录ID的变量名，使用_id, _ids后缀
* One2Many, Many2Many字段，应该使用_ids后缀
* Many2One字段，应该使用_id后缀


#### 方法名
* 计算方法： _compute_\<field-name\>
* 搜索方法： _search_\<field-name\>
* 默认方法： _default_\<field-name\>
* 选择方法： _selection_\<field-name\>
* onchange方法： _onchange_\<field-name\>
* 约束方法： _check_\<constraint-name\>
* 动作方法： action_\<name\>


#### 模型类中属性排序
1. 私有属性(_name, _description, _inherit)
1. 默认方法
1. 字段声明
1. 计算方法、搜索方法、选择方法、约束方法、onchange方法
1. CRUD方法
1. 动作方法
1. 其他方法