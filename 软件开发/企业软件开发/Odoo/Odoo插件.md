

# Odoo插件

[TOC]

## 分类
* base  基础
* product 定义产品相关信息
* sale   销售
* sale_management
* sale_purchase



## 模型

### product.template 产品模板
#### product 定义
* name 产品名字
* description 描述信息
* description_purchase 采购描述信息
* description_sale 销售描述信息
* type 类型
* categ_id  分类信息，指向product.category

* price 产品价格
* list_price 销售价格
* 1st_price 公开价格
* standard_price 成本价格

* sale_ok 是否用于销售
* purchase_ok 是否用于采购

* company_id 所属公司，指向res.company

* image 图片
* image_medium
* image_small

#### sale 增加
* service_type 服务类型
* sales_count 销售数量
* invoice_policy 开票策略


### res.partner 联系人
#### base 定义
* name 名字
* display_name 显示名字
* date 日期
* is_company 是否是公司
* company_type 公司类型：person, company
* type 地址类型：contact, invoice, delivery

* parent_id 所属公司
* user_id 指向res.users 
* user_ids
* customer 是否是顾客
* supplier 是否是制造商



### res.company 公司
* name 名字

* parent_id 指向res.company
* child_ids 指向res.company

* partner_id 指向res.partner,多对一
* user_ids 指向res.users，多对多


### res.users 用户
#### base 定义
* 通过委托继承于res.partner
* partner_id 指向res.partner表，多对一
* login 登录名
* password 登录密码

* company_id 关联res.company，多对一
* company_ids 关联res.company, 多对多