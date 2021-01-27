

# Odoo插件

[TOC]

## 分类
* base  基础
* contacts 联系人
* portal  门户

* product 产品信息
* purchase 采购
* sale   销售
* stock 库存

## base
* 定义基础业务模型：res_xxx
* 定义软件数据模型：ir_xxx

### 模型
#### res.partner 联系人
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

#### res.company 公司
* name 名字

* parent_id 指向res.company
* child_ids 指向res.company

* partner_id 指向res.partner,多对一
* user_ids 指向res.users，多对多

#### res.users 用户
* 通过委托继承于res.partner
* partner_id 指向res.partner表，多对一
* login 登录名
* password 登录密码

* company_id 关联res.company，多对一
* company_ids 关联res.company, 多对多


## 核心功能 -- 进销存

### 模型
#### product.template 产品模板 
##### 字段
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

##### sale继承
* service_type 服务类型
* sales_count 销售数量
* invoice_policy 开票策略


#### product.product 产品
* 继承了product.template


#### sale.order 销售订单
* 定义一张销售订单，包含多个订单项

##### 字段
* name 名字
* origin 原始文档
* state 状态

* user_id 销售人员，指向res.users
* partner_id 顾客， 指向res.partner 

* order_line 订单内容， 指向sale.order.line


#### sale.order.line 销售订单项
* 定义一张销售订单项，要销售的一件商品

##### 字段
* order_id 订单ID
* name 名字

* product_id 产品ID，指向product.product


#### purchase.order 采购订单
##### 字段
* name 名字
* state 状态

* partner_id 制造商，指向res.partner
* order_line 订单内容，指向purchase.order.line


#### purchase.order.line 采购订单内容
##### 字段
* order_id 订单ID
* name 名字

* product_id 产品ID，指向product.product



## 代发货
### 组件关系
* stock_dropshipping 代发货
    * sale_purchase_stock 按订单生产(MTO)
        * sale_purchase 基于服务外包的销售
        * sale_stock   关联销售和库存
        * purchase_stock 关联采购和库存


### sale_purchase
* 增加SO和PO之间的关联
* 通过SO直接生成PO

#### 继承模型
##### product.template
* service_to_purchase 是否自动生成PO

#### purchase.order 
* sale_order_count 源销售订单的个数

#### purchase.order.line 
* sale_line_id 关联到 sale.order.line 表
    * 将采购订单项 和 销售订单项 关联起来，一个SOL可以对应多个POL
* sale_order_id 由关联的销售订单项，获得销售订单号

##### sale.order
* purchase_order_count 采购订单个数，通过订单项动态计算

##### sale.order.line 
* purchase_line_ids 关联到 purchase.order.line 表
* purchase_line_count 所有采购订单项的个数
* ceate() 在创建订单项时，创建对应的采购订单、采购订单项



## portal
* 定义了门户用户登录后的界面


### 逻辑
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