

# Odoo插件

[TOC]

## 分类
* base  基础
* portal  门户
* product 产品信息
* purchase 采购
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

* price 最终销售价格，计算字段，基础是list_price
* list_price 基准价格
* 1st_price 公开价格，和list_price等价
* standard_price 采购是的成本价格（用于标准计价法）

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



### product.product 产品
#### product 定义
* 继承了product.template




### sale.order 销售订单
#### sale 定义
* name 名字
* origin 原始文档
* state 状态

* user_id 销售人员，指向res.users
* partner_id 顾客， 指向res.partner 

* order_line 订单内容， 指向sale.order.line


### sale.order.line 销售订单内容
* order_id 订单ID
* name 名字

* product_id 产品ID，指向product.product


### purchase.order 采购订单
#### purchase 定义
* name 名字
* state 状态

* partner_id 制造商，指向res.partner

* order_line 订单内容，指向purchase.order.line


### purchase.order.line 采购订单内容
#### purchase 定义
* order_id 订单ID
* name 名字

* product_id 产品ID，指向product.product


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



## 逻辑

### portal
#### 登录主页面
* 路由：xxx/my 或 xxx/my/home (controllers/portal.py)
* 显示模板：portal_my_home (views/portal_templates.xml)

##### 基础结构
1. 通用元素，调用portal_layout
2. 文档区域 class为 o_portal_my_home
    1. 文本 Documents
    1. 列表项 class为 o_portal_docs

##### 继承结构
sale插件会继承该模板，增文档列表项

* 模板： portal_my_home_sale （views/sale_portal_templates.xml）
* 扩展点：o_portal_docs 内部
* 扩展方式：使用portal.portal_docs_entry 增加列表项（报价单，销售订单）