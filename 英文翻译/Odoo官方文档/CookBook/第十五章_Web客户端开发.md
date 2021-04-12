


# 第十五章_Web客户端开发


[TOC]


## 概述

### 框架
* Odoo14包含了两个不同的框架来实现后端的界面
    * 第一个是 基于控件的老框架
    * 第二个是 基于组件的新框架（称为OWL）
* 虽然有了新框架OML，但是Odoo不会在所有地方都使用该框架
* 大部分web客户端都依然使用老框架来实现



## 创建自定义控件

### 总述
* 正如在第九章看见的那样，我们使用控件来用不同的格式显示特定的数据
    * 例子：我们使用widget='image'来将二进制字段显示为图片
* 为了说明如何创建你自己的控件，我们会创建一个用户可以选择整型的控件
    * 该控件不使用输入框，而是显示一个颜色选择器，用户可以选择一个颜色编码


### 如何操作
1. 添加一个static/src/js/field_widget.js文件
    ```js
    odoo.define('my_field_widget', function(require) {
        var AbstractField = require('web.AbstractField')
        var fieldRegistry = require('web.field_registry')
        //通过继承AbstractField来创建控件
        var colorField = AbstractField.extend({
            className: 'o_int_colorpicker',  //设置CSS类
            tagName: 'span',  //根元素标识
            supportedFieldTypes: ['integer'],
            //捕获js事件
            events: {
                'click.o_color_pill':'clickPill', 
            },
            //初始化组件
            init: function() {
                ....
            },
            //覆盖父类函数，设置DOM元素
            _renderEdit: function() {
                ...
            },
            _renderReadonly: function() {
                ...
            },
            //定义事件处理函数
            clickPill: function() {
                ...
            }
        });
        //注册控件
        fieldRegistry.add('int_color', colorField);
        //使其对其他插件有效
        return { 
            colorField: colorField,
        }
    });
    ```
1. 添加static/src/scss/field_widget.scss文件
    ```css
    .o_int_colorpicker {
        .o_color_pill {
            ...
        }
    }
    ```
1. 在views/templates.xml中注册js和scss文件
1. 在library.book模型中加入integer字段
    ```python
    color = fields.integer()
    ```
1. 在视图中加入控件
    ```xml
    <group>
        <field name="color" widget="int_color"/>
    </group>
    ```


### 工作原理

#### 控件生命周期
##### init()
* 控件的构造函数，用于初始化控件
* 当控件初始化后，该函数会第一个被调用

##### willStart()
* 当控件初始化完成并且在被加入到DOM中过程中，会调用该方法
* 该方法被用于初始化异步数据
* 该方法还支持返回一个从super()中获取到的延迟对象

##### start()
* 当控件完成渲染后但是还没有被加入DOM时，会调用该方法
* 该方法适合于完成后渲染任务
* 该方法假定会返回一个延迟对象
* 可以通过存取this.$el来获取渲染后的元素

##### destroy()
* 当控件被销毁后，会调用该方法
* 该方法被用于基础的清除资源操作：像解绑事件


##### 要点
* 控件的基础类是Widget，由web.Widget定义


#### 步骤说明
##### 步骤1
* 添加了三个属性
    * className: 定义控件根元素的类名
    * tagName: 定义根元素类型
    * supportedFieldType: 定义了控件所支持的字段类型
* 映射了控件的事件
    * 键：使用事件名或则CSS选择器，两者使用空格分隔
    * 值：控件方法的名字
    * 当事件被触发时，对应的方法会被自动调用
* 覆盖了两个方法：_renderEdit, _renderReadonly
    * _renderEdit: 当控件处于编辑模式时，该方法会被调用
    * _renderReadonly: 当控件处理只读模式时，该方法会被调用



### 更多
* web.mixins命名空间定义了好几个非常有用，在开发控件时不容错过的的mixin类
* 在这个例子中你已经在使用这些mixin类了
    * AbstractField继承于Widget类
    * Widget类继承了两个mixin类
    * EventDispatcherMixin：提供了简单的界面来关联事件处理器并触发它们
    * ServicesMixin：提供了RPC调用和动作的功能


### 要点
* 当我们要覆盖一个方法时，经常需要学习基类来了解该函数会返回什么
* 一个经常会出现bug的情况就是忘记了返回超类的延时对象，从而在异步操作时发生错误





## 使用客户端QWeb模板

### 总述
* 在js代码中创建HTML代码是一个坏的编程习惯，你应该只创建最小量的DOM元素
* 幸运的是在客户端也可以使用模板引擎，更幸运的是客户端的模板引擎使用服务端模板相同的语法


### 如何操作
1. 导入web.core和qweb
    ```js
    odoo.define('my_field_widget', function(require) {

        var core = require('web.core')
        var qweb = core.qweb
    })
    ```
1. 修改_renderEdit方法，使用模板引擎
    ```js
    _renderEdit: function() {
        this.$el.empty();
        var pills = qweb.render('FieldColorPills', {widget:this});
        this.$el.append(pills);
    },
    ```
1. 添加模板文件static/src/xml/qweb_template.xml
    ```xml
    <templates>
        <t t-name="FieldColorPills"> 
            <t t-foreach="widget.totalColors" t-as="pill_no">
               <span t-attf-class="o_color_pill o_color_#{pill_no}
                #{widget.value === pill_no and 'active' or ''}"
                t-att-data-val="pill_no"/> 
            </t>
        </t>
    </templates>
    ```
1. 在manifest中注册模板
    ```json
    "qweb": [
        'static/src/xml/qweb_template.xml',
    ]
    ```


### 工作原理

#### 引擎的不同点
* 客户端的引擎是QWeb的js实现，是不同于服务端上的Python实现的
    * 你无法获取、浏览记录或则环境
    * 你仅仅能够获取传入给qweb.render函数的参数
* 在例子中，我们使用widget键传入了当前的对象
    * 你可以和控件中的js代码进行整合，模板可以存取属性或者函数

#### 注册
* 因客户端QWeb与QWeb视图没有关系，所以需要不同的机制来让客户端知道这些模板
* 在插件manifest中的qweb区域中加入模板

#### 注意
* 如果你不想在manifest中列出QWeb模板文件，你可以使用xmlDependencies键来延时加载模板
    * 模板只会在控件初始后才会被加载



### 更多
* 我们在这里使用QWeb需要化更多精力的的原因是多方面的，其中一个就是客户端QWeb和服务端QWeb的第二个大的不同点
* 在客户端，我们无法使用xpath表达式，需要使用jquery选择和操作符
* 例子：在其他模块给我们的控件增加图标
    ```xml
    <t t-extend="FieldColorPills">
        <t t-jquery="span" t-operation="prepend">
            <i class="fa fa-user"/>
        </t>
    </t>
    ```

#### 操作符值
* append：t元素的内容会被附加在匹配元素的内容上
* before: t元素的内容会被加入到匹配元素的内容的前面
* after: t元素的内容会被加入到匹配元素的内容的后面
* inner：t元素的内容会替换到部分匹配元素的内容
* repalce：t元素的内容会替换到全部匹配元素的内容
* attribute：给匹配元素设置属性

#### 另一个不同点
* 客户端QWeb是没有命名空间的
* 所以在选择模板名字时，需要让名字成为所有你安装插件中的唯一的名字
    * 这就是开发者会尽量使用更长名字的原因




## 向服务器进行RPC调用

### 总述
* 迟早你的控件就需要从服务器上查询一些数据
* 例子：在颜色药丸上增加提示信息，当鼠标悬停在药丸上时，提示信息会显示出和该颜色相关的书的个数

### 如何操作
1. 增加willStart方法
    ```js
    willStarT: function() {
        var self = this;
        this.colorGroupData = {};
        var colorDataPromise = this._rpc({
            model: this.model,
            method: 'read_group',
            domain: [],
            fields: ['color'],
            groupBy: ['color],
        }).then(function(result) { 
            _.each(result, function(r) {
                self.colorGroupData[r.color] = r.color_count;
            });
        });
        return Promise.all([this._super.apply(this, arguments)], colorDataPromise)
    }
    ```
1. 更新_renderEdit方法
    ```js
    _renderEdit: function() {
        ...
        this.$el.find('[data-toggle="tooltip"]').tooltip();
    },
    ```
1. 更新FieldColorPills模板
    ```xml
    <t t-name="FieldColorPills">
        <span ...
            data-toggle="tooltip"
            data-placement="top"
            t-attf-title="This color is used in #{widget.colorGroupData[pill_no] or 0} books"
        />
    </t>
    ```


### 工作原理
#### willStart
* 该函数会在渲染前被调用，更重要的是，它会返回一个必须在渲染开始前要解决的Promise对象
* 在我们的例子中，我们需要在渲染发生前运行一个异步动作

#### _rpc
* 当要处理数据存取时，我们依赖于ServicesMixin类提供的_rpc函数
* 该函数允许我们调用模型上的任何公开函数，像：search,read,write或read_group

#### 步骤1
* 我们使用一个RPC调用来触发library.book模型上的read_group方法
* 我们基于color字段来分组数据，所以RPC返回的书数据是color分组后的，并加入了color_count的统计键
* 我们在colorGroupData中映射了color和color_count，以便在QWeb模板中使用

* 在最后一行，我们使用super解析了willStart，并且RPC调用使用了$.when
    * 所以渲染只会在数据获取后，并且在之前所有的异步操作完成之后

#### 步骤3
* 我们将colorGroupData设置给需要显示tooltip的属性
* 在willStart方法中，我们使用this.colorGroupData分配了一个color映射
    * 在QWeb模板中，使用widget.colorGroupData来访问它们

#### 注意
* 我们可以在控件的任何地方使用_rpc
* 注意这是一个异步调用，所以需要正确的管理延时对象，以获取期望的结果


### 更多
#### AbstractField
该类包含一些有意思的属性

* this.model：引用了当前模型的名字（在例子中，就是library.book）
* this.field：包含了模型field_get()的粗略输出，即控件要显示的字段
    * 该方法可以获取当前字段的所有信息
    * 对于x2x字段，该函数会返回共同模型或者domain的信息
    * 你可以使用该属性查询到字段的字符串、大小、以及其他信息
* this.nodeOptions：包含了在form视图定义的options属性传入的数据
    * 该数据已经按JSON进行了解析，你将其作为一个对象来访问它


    


## 创建视图

### 总述
* 本节将创建一个称为m2m_group的视图，用于按组来显示记录。将记录划分到不同的组中

### 如何操作
1. 增加一个新的视图类型
    ```python
    class View(models.Model) :
        _inherit = 'ir.ui.view'
        type = fields.Selection(selection_add=[('m2m_group', 'M2m Group')])
    ```
1. 增加一个新的视图模式
    ```python
    class ActWindowView(models.Model) :
        _inherit = 'ir.actions.act_window.view'
        view_mode = fields.Selection(selection_add=[('m2m_group', 'M2m Group')], 
                                     ondelete=('m2m_group':'cascade'))
    ```
1. 给base模型增加新方法
    ```python
    class Base(models.AbstractModel) :
        _inherit = 'base'

        @api.model
        def get_m2m_group_date(self, domain, m2m_field) :
            records = self.search(domain)
            result_dict = {}
            for record in records :
                for m2m_record in record[m2m_field] :
                    if m2m_record.id not in result_dict :
                        result_dict[m2m_record.id] = {
                            'name': m2m_record.display_name,
                            'children': [],
                            'model': m2m_record._name
                        }
                    result_dict[m2m_record.id]['children'].append({
                        'name': record.display_name,
                        'id': record.id,
                    })
            return result_dict
    ```
1. 增加客户端模型文件 /static/src/js/m2m_group_model.js
    ```js
    odoo.define('m2m_group.Model', function(require) {
        var AbstractModel = require('web.AbstractModel')
        var M2mGroupModel = AbstractModel.extend({
            _get: function() { 
                return this.data;
            },
            __load: function (params) {
                this.modelName = params.modelName;
                this.domain = params.domain;
                this.m2m_field = params.m2m_field;
                return this._fetchData();
            },
            __reload: function(handle, params) {
                if('domain' in params) {
                    this.domain = params.domain;
                }
                return this._fetchData();
            },
            _fetchData: function() {
                var self = this;
                return this._rpc({
                    model: this.modelName,
                    method: 'get_m2m_group_data',
                    kwargs: {
                        domain: this.domain,
                        m2m_field: this.m2m_field
                    }
                }).then(function (result) {
                    self.data = result;
                });
            },
        });
        return M2mGroupModel;        
    });
    ```
1. 增加客户端控制器文件 /static/src/js/m2m_group_controller.js
    ```js
    odoo.define('m2m_group.Controller', function(require) {
        var AbstractController = require('web.AbstractController');
        var core = require('web.core');
        var qweb = core.qweb;
        var M2mGroupController = AbstractController.extend({
            custom_events: _.extend({}, AbstractController.prototype.custom_events, {
                'btn_clicked': '_onBtnClicked',
            }),
            renderButtons: function ($node) {
                if ($node) {
                    this.$buttons = $(qweb.render('ViewM2mGroup.buttons'));
                    this.$buttons.appendTo($node);
                    this.$buttons.on('click', 'button', this._onAddButtonClick.bind(this));
                }
            },
            _onBtnClicked: function (ev) {
                this.do_action({
                    type: 'ir.actions.act_window',
                    name: this.title,
                    res_model: this.modelName,
                    views: [[false, 'list'], [false, 'form']],
                    domain: ev.data.domain,
                });
            },
            _onAddButtonClick: function (ev) {
                this.do_action({
                    type: 'ir.actions.act_window',
                    name: this.title,
                    res_model: this.modelName,
                    views: [[false, 'form']],
                    target: 'new'
                });
            },
        });
        return M2mGroupController;
    });
    ```
1. 增加客户端渲染器文件/static/src/js/m2m_group_renderer.js
    ```js
    odoo.define('m2m_group.Renderer', function(require) {
        var AbstractRenderer = require('web.AbstractRenderer')
        var core = core.qweb;
        var M2mGroupRenderer = AbstractRenderer.extend({
            events: _.extend({},AbstractRenderer.prototype.custom_events, {
                'click.o_primay_button': '_onClickButton',
            }),
            _render: function() {
                var self = this;
                this.$el.empty();
                this.$el.append(qweb.render('ViewM2mGroup', {
                    'groups': this.state,
                }));
                return this._super.apply(this, arguments);
            },
            _onClickButton: function(ev) {
                ev.preventDefault();
                var target = $(ev.currentTarget);
                var group_id = target.data('group');
                var children_ids = 
                        _.map(this.state[group_id].children, function (group_id) {
                            return group_id.id;
                        });
                this.trigger_up('btn_clicked', {
                    'domain': [['id', 'in', children_ids]]
                });
            },
        });
        return M2mGroupRenderer;
    });
    ```
1. 增加客户端视图文件/static/src/js/m2m_group_view.js
    ```js
    odoo.define('m2m_group.View', function (require) {
        var AbstractView = require('web.AbstractView');
        var view_registry = require('web.view_registry');

        var M2mGroupController = require('m2m_group.Controller');
        var M2mGroupModel = require('m2m_group.Model');
        var M2mGroupRenderer = require('m2m_group.Renderer');

        var M2mGroupView = AbstractView.extend({
            display_name: 'Author',
            icon: 'fa-id-card-o',
            config: _.extend({}, AbstractView.prototype.config, {
                Model: M2mGroupModel,
                Controller: M2mGroupController,
                Renderer: M2mGroupRenderer,
            }),
            viewType: 'm2m_group',
            searchMenuTypes: ['filter', 'favorite'],
            accesskey: "a",
            init: function (viewInfo, params) {
                this._super.apply(this, arguments);
                var attrs = this.arch.attrs;
                if (!attrs.m2m_field) {
                    throw new Error('M2m view has not defined "m2m_field" attribute.');
                }
                // Model Parameters
                this.loadParams.m2m_field = attrs.m2m_field;
            },
        });
        view_registry.add('m2m_group', M2mGroupView);
        return M2mGroupView;
    });
    ```
1. 增加视图QWeb模板文件/static/src/xml/qweb_template.xml
    ```xml
    <t t-name="ViewM2mGroup">
        ...
    </t>
    ```
1. 将所有js文件加入后端资源中
1. 给library.book模型增加新视图
    ```xml
    <record id="library_book_view_author" model="ir.ui.view">
        <field name="name">Library Book Author</field>
        <field name="model">library.book</field>
        <field name="arch" type="xml">
            <m2m_group m2m_field="author_ids" color_field="color"></m2m_group>
        </field>
    </record>
    ```
1. 给Book动作增加m2m_group类型
    ```xml
    <field name="view_mode">tree,m2m_group,form</field>
    ```

### 显示效果

![](https://gitee.com/cc12703/figurebed/raw/master/img/20210414112956.png)




### 工作原理
#### 步骤3
* 在base上增加方法可以保证该方法对于所有模型都有效
* 该方法可以从js视图中通过RPC被调用到
* 需要创建两个参数：domain和m2m_field
    * domain参数的值，会是由搜索视图domain和动作domain的合并生成的
    * m2m_field参数，用于指定使用哪个字段进行记录分组


#### 视图结构
* js视图由多个部分组成：视图View, 模型Model, 渲染器renderer, 控制器controller
* 由于View在Odoo代码中已经有了别的含义，所有MVC(Model，View, Controller)变成了MRC(Model, Renderer, Controller)
* 总体上说，视图由模块、渲染器、控制器按MVC层次组合在一起

##### 图示
![](https://gitee.com/cc12703/figurebed/raw/master/img/20210414114447.png)


##### 模型
* 模型的作用是来保存视图的状态
* 发送RPC请求给服务器获取数据，并传递数据给控制器和渲染器
* 当视图初始化完成后，会调用__load()来获取数据
* 当搜索条件变化，视图需要新的状态时，__reload()会被调用
* 当控制器需要获取模型的状态时，__get()会被调用

##### 控制器
* 控制器的作用协调模型和渲染器
* 当渲染器发生动作时，渲染器会将信息传送给控制器并执行对应的动作，有时候还会调用模型的方法
* 此外，控制器还管理着控制面板中的按键
* 例子中，我们增加了一个按键用于添加新记录
    * 需要覆写AbstractController的renderButtons()方法
* 我们还注册了custom_events用来当作者卡中的按键被点击后，渲染器可以触发事件给控制器来指定动作

##### 渲染器
* 渲染器的作用管理视图的DOM元素，每个视图都使用不同的方式来渲染数据
* 在渲染器中，可以通过state变量获取到模型状态，调用render()方法来渲染
* 例子中，我们使用当前状态来渲染ViewM2mGroup的QWeb模板，来显示视图
* 例子中，我们映射了js事件到自定义事件
    * 我们给卡片中的按键绑定了点击事件，一旦用户点击该按键，渲染器就会触发btn_clicked事件给控制器

###### 要点
* 注意events和custom_events是不同的。
    * events是标准的js事件
    * custom_events来自于Odoo的js框架，该事件可以通过trigger_up方法触发


##### 视图
* 视图的作用是使用基本元素来构建js视图
    * 基本元素：字段、上下文、视图架构、参数信息
* 视图会初始化控制器、渲染器和模型，将它们按MVC层次连接起来
* 视图会给模型、渲染器、控制器设置需要的参数
    * loadParams 模型参数
    * controllerParams 控制器参数
    * renderParams 渲染器参数




## 更多
* 如果你不想引入一个新的视图类型，而是想对视图进行少量修改，你可以对视图使用js_class
* 例子：如果你想让一个视图看起来向看板一样，可以参考一下代码
    ```python
    var CustomDashboardRenderer = kanbanRenderer.extend({
        ...
    });
    var CustomDashboardModel = KanbanModel.extend({
        ...
    });
    var CustomDashboardController = KanbanController.extend({
        ...
    });
    var CustomDashboardView = KanbanView.extend({
        config: _.extend({},KanbaView.prototype.cofig, {
            Model: CustomDashboardModel,
            Renderer: CustomDashboardRenderer,
            Controller: CustomDashboardController,
        })
    });
    var viewRegistry = require('web.view_registry');
    viewRegistry.add('my_custom_view', CustomDashboardView);
    ```
    ```xml
    <field name="arch" type="xml">
        <kanban js_class="my_cutom_view">
            ...
        </kanban>
    </field>
    ```