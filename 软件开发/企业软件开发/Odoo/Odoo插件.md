

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
    * purchase_stock 采购相关
    * sale_stock 销售相关

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

### 数据库表
* stock_warehouse 仓库
* stock_location 库位
* stock_picking 分拣单（出库、入库、内部移动）
* stock_move 移动记录表，每次分拣就会触发一次移动
* stock_quant 商品库存表，存放商品数量
* stock_inventory 库存盘点
* stock_inventory_line 库存盘点明细
* stock_production_lot 序列号（产品批次）

* procurement_rule 补货规则
* procurement_order 补货单



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


#### stock.warehouse 仓库
* name 仓库名
* company_id 所属公司
* partner_id 仓库地址
* code 识别码

* reception_steps 入库步骤
* delivery_steps 出库步骤

* wh_input_stock_loc_id  输入库位
* wh_qc_stock_loc_id  质检库位
* wh_output_stock_loc_id    输出库位
* wh_pack_stock_loc_id    打包库位

* pick_type_id  拣货类型
* pack_type_id 打包类型
* out_type_id  出库类型
* in_type_id 入库类型


#### stock.location 仓库库位
* name 库位名字
* complete_name 库位全名
* barcode 二维码
* usage 用途（供应商、内部、客户、转发、盘点）

* location_id 父库位
* child_ids 子库位

* quant_ids  库存信息, 指向stock.quant


#### stock.quant  库存信息
* product_id  产品，指向product.product
* product_tmpl_id 产品模板，指向product.template
* product_uom_id 度量单位

* lot_id 产品串号, 指向stock.production.lot
* package_id  包装，指向stock.quant.package
* owner_id    拥有者，指向res.partner

* in_date 入架时间

* quantity 总数量
* inventory_quantity 盘点数量
* reserved_quantity  保留数量
* available_quantity 有效数量

##### stock.quant.package 库存包装
* name  名字
* packaging_id  包装类型，指向product.packaging


#### stock.picking 仓库分拣单
* name 信息
* note 注释
* origin  源文档

* move_type 发货策略（direct, one）
* picking_type_id 分拣类型，指向stock.picking.type
* group_id 补货组

* partner_id 联系人，指向res.partner
* user_id 负责人，指向res.users
* owner_id  拥有人，指向res.partner

* location_id 源库位，指向stock.location
* location_dest_id 目标库位，指向stock.location


## 代发货
### 组件关系
* stock_dropshipping 代发货
    * sale_purchase_stock 按订单生产(MTO)
        * sale_purchase 基于服务外包的销售
        * sale_stock   关联销售和库存
        * purchase_stock 关联采购和库存

### 规则数据
* 在stock.location.rule中生成一条记录 route_drop_shipping
* 在stock.rule表中为每家公司生成一条记录


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



## 仓库

### 规则逻辑
#### 订单行启动规则  
* 位置：sale-stock:SaleOrderLine->_action_launch_stock_rule()
* 逻辑：
    1. 生成需求组ID
    1. 生成需求信息
    1. 运行规则选择

#### 规则选择
* 位置：stock:ProcurementGroup->run()
* 逻辑：
    1. 生成查询domain _get_rule_domin()
    1. 查找规则 _search_rule
    1. 获取到规则对应的动作 action
    1. 调用StockRule类的_run_\<action\>方法

#### 触发动作
* 移库：stock:StockRule._run_poll()
* 采购：purchase_stock:StockRule._run_buy()
* 制造：mrp:StockRule._run_manufacture()    


#### 生成采购单
* 位置：purchase_stock:StockRule._run_buy()
* 逻辑：
    1. 确定供应商 supplier
        * 通过product的_select_seller(),_prepare_sellers()确定
    1. 生成采购单 
        * 通过_prepare_purchase_order()生成数据
        * 通过调用模型的create()生成订单
    1. 生成采购单行
        * 通过PurchaseOrderLilne._prepare_purchase_order_line()生成数据
        * 通过调用模型的create()生成订单行



### 接收单

#### 创建接收单
* 位置：purchase_stock:PurchaseOrder.button_approve()
* 逻辑：
    1. 调用父类的button_approve()
    1. 调用_create_picking()创建接收单
        1. 调用_prepare_picking()生成数据
        1. 调用模型的create()生成单子