

# 第十六章_OWL

[TOC]


## 概述
* Odoo 14引入了一个新的js库，称为OWL(Odoo Web Library)
* OWL是一个基于组件的界面框架，使用QWeb模板进行构建
* 相比于Odoo老的控件系统，OWL很快速，并且提供了大量的新特征


### 技术要求
* OWL组件需要使用ES6类来定义



## 创建组件

### 如何操作
1. 添加/static/src/js/component.js文件，定义一个模块命名空间
    ```js
    odoo.define('my.component', function(require) {
        ...
    });
    ```
1. 增加/view/templates.xml文件，加载组件
    ```xml
    <template id="assets_end" inherit_id="web.assets_backend">
        <xpath expr="." position="inside"> 
            <script src="/static/src/js/component.js" type="text/javascript" />
        </xpath>
    </template>
    ```
1. 在component.js中定义OWL工具库
    ```js
    const { Component } = owl;
    const { xml } = owl.tags;
    ```
1. 在component.js中增加基础模板
    ```js
    class myComponent extends Component {
        static template = xml`
            <div class="xxx">    
                <b> Welcome to Odoo </b>
            </div>`
    }
    ```
1. 在component.js中增加初始化操作
    ```js
    owl.utils.whenReady().then(() => {
        const app = new MyComponent();
        app.mount(document.body);
    })
    ```


### 工作原理
#### 步骤1，2
* 添加js文件并加载到后端assets中

#### 步骤3
* 从OWL中初始化一个变量
* OWL中所有的工具类都位于一个全局变量下，名为owl
* 例子中，我们定义了Component，从owl.tags中定义了xml
    * Component是OWL组件的主类，所有组件都需要从该类扩展

#### 步骤4
* 通过扩展Component类来创建自己的组件MyComponent
* 使用语法xml\`...`定义了QWeb模板，该语法称为内嵌模板
    * 你也可以从独立文件中加载模板
    * 内嵌模板无法支持继承的修改和翻译

#### 步骤5
* 实例化MyCompoment组件，并将其加入页面中
* OWL组件是一个ES6的类，所以可以从new关键字来创建对象
* 使用mount()方法将组件加入页面
* whenReady()回调，会保证在使用OWL前所有的OWL功能已经加载完成



### 更多
* OWL是一个独立的库，在Odoo中作为一个外部js库来加载
* OWL位于 https://github.com/odoo/owl
* 例子位于 https://odoo.github.io/owl/playground/




## 管理用户动作

### 总述
* 为了能使用户界面有交互能力，组件需要处理用户动作，像：点击、悬停、表格提交


### 如何操作
1. 更新QWeb模板，增加图标来移除组件
    ```js
    static template = xml`
        <div class="xxxx">
            <b> xxxx </b>
            <i class="xxxx" style="cursor:pointer;"
            t-on-click="onRemove"></i>
        </div>
    `
    ```
1. 在组件中增加onRemove方法
    ```js
    class MyComponent extends Component {
        onRemove(ev) {
            this.destroy();
        }
    }
    ```


### 工作原理
#### 步骤1
* 给组件增加了一个移除图标，添加了一个t-on-click属性
* 该属性用于绑定一个点击事件
    * 属性的值是组件的一个方法
    * 例子：t-on-click="onRemove"，当用户点击移除图标时，onRemove方法会被调用
* 定义事件的语法：t-on-\<事件名字>=“\<组件中的方法名>”


#### 步骤2
* 增加了onRemove方法，方法中调用了destroy()方法，该方法将组件从DOM中移除
* onRemove方法中，我们接收到一个事件对象
* destroy()是OWL组件的默认方法之一


### 更多
* 事件处理并不限制于DOM事件，你可以使用自定义事件
* 例子：如果你触发了my-custom-event事件，你可以使用t-on-my-custom-event来捕获该自定义事件



## 使组件响应化

### 总述
* OWL是一个强大的框架，支持基于hook的界面自动更新
* 借助更新hook，当组件的内部数据变化时，组件的界面会自动进行更新


### 如何操作
1. 更新模板，增加两个按键和一个字符串
    ```js
    static template = xml`
        <div class="xxx">   
            <i class="xxx" style="cursor:pointer;" t-on-click="onPreious"/>
            <b t-esc="messageList[Math.abs(state.currentIndex%4)]" />
            <i class="xxx" style="cursor:pointer;" t-on-click="onNext"/>
            <i class="xxx" style="cursor:pointer;" t-on-click="onRemove"/>
        </div>
    `
    ```
1. 在组件中，导入useState钩子
    ```js
    const { Component, useState } = owl;
    ```
1. 在组件中，增加初始化函数
    ```js
    constructor() {
        super(...arguments);

        this.messageList = ['xxx', 'xxxx', 'xxxx'];

        this.state = useState({ currentIndex: 0})
    }
    ```
1. 在组件中，增加事件处理函数
    ```js
    onNext(ev) {
        this.state.currentIndex++;
    }
    onPreious(ev) {
        this.state.currentIndex++;
    }
    ```


### 工作原理
#### 步骤1
* 从消息列表中渲染消息字符串，基于currentIndex的值来选择消息
* 增加两个箭头图标，使用t-on-click绑定点击事件

#### 步骤2，3
* 导入了useState钩子
    * 该钩子用于处理组件状态
* 增加了初始化函数
    * 该函数在组件实例化时会被调用
* 在初始化函数中，增加了消息列表，使用useState添加了状态变量
* 该钩子会是组件响应化
    * 当状态发送变化，界面会基于新的状态来刷新


#### 重点
* 定义钩子是有一个规则：必须在构造函数中进行定义
* 更多钩子：https://github.com/odoo/owl/blob/master/doc/reference/hooks.md





## 理解组件的生命周期

### 如何操作
1. 在构造函数中输出日志
    ```js
    constructor() {
        console.log('CALLED:> constructor');
    }
    ```
1. 增加回调函数
    ```js
    async willStart() {
        console.log('CALLED:> willStart');
    }
    mounted() {
        console.log('CALLED:> mounted');
    }
    willPatch() {
        console.log('CALLED:> willPatch');
    }
    patched() {
        console.log('CALLED:> patched');
    }
    willUnmount() {
        console.log('CALLED:> willUnmount');
    }
    ```

#### 日志输出结果
* 操作流程：重启系统、更新组件、
> CALLED:> constructor
> CALLED:> willStart
> CALLED:> mounted
> CALLED:> willPatch
> CALLED:> patched
> CALLED:> willUnmount



### 工作原理
#### constructor
* 该函数是生命周期中首先被调用的
* 当你初始化组件时会被调用
* 该函数用于设置组件的初始状态

#### willStart
* 该函数在组件构造之后，元素渲染之前被调用
* 该函数是一个异步方法，可以执行异步操作

#### mounted
* 该函数在组件渲染被加入到DOM之后被调用

#### willPatch
* 该函数在组件状态改变时被调用
* 该函数在元素基于新状态重新渲染之前被调用

#### patched
* 该函数工作方式类似于willPatch，也是在组件状态改变时被调用
* 不同于willPathc，该函数在元素基于新状态重新渲染之后被调用

#### willUnmount
* 该函数只在元素被从DOM中移除之前被调用

#### 总结
* mounted, willUnmount可以用于绑定、解绑事件监听器


### 更多
#### willUpdateProps
* 该函数用于子组件的情况下
* OWL会使用props参数传入父组件的状态，当props改变时，该函数会被调用
* 该函数是一个异常方法，可以执行异步操作





## 添加字段到form视图

### 如何操作
1. 在library.book模型中增加字段
    ```python
    color = fields.Integer()
    ```
1. 在form视图中增加控件
    ```xml
    <field name="color" widget="int_color"/>
    ```
1. 增加QWeb模板，/static/src/xml/qweb_template.xml
    ```xml
    <templates>
        <t t-name="OWLColorPill" owl="1">
            <span t-attf-class="o_color_pill o_color_{{props.pill_no}} {{props.active and 'active' or ''}}"
                t-att-data-val="props.pill_no" t-on-click="pillClicked"
                t-attf-title="The color is used in {{props.book_count or 0}} books."/>
        </t>
        <span t-name="OWLFieldColorPills" owl="1" class="xxxx" t-on-color-updated="colorUpdated">
            <t t-foreach="totalColors" t-as="pill_no">
                <ColorPill t-if="mode === 'edit' or value == pill_no" 
                        pill_no='pill_no' active='value == pill_no'
                        book_count="colorGroupData[pill_no]"/>
            </t>
        </span>
    </templates>
    ```
1. 注册QWeb文件
    ```json
    'qweb': [
        'static/src/xml/qweb_template.xml',
    ]
    ```
1. 在static/src/scss/field_widget.scss中增加SCSS
1. 增加static/src/js/field_widget.js文件
    ```js
    odoo.define('my_field_widget', function(require) {
        const { Component} = owl;
        const AbstractField = require('web.AbstractFieldOwl');
        const fieldRegistry = require('web.field_registry_owl');
    })
    ```
1. 增加ColorPill组件
    ```js
    class ColorPill extends Component {
        static template =  'OWLColorPill';
        pillClicked() {
            this.trigger('color-updateed', {val: this.props.pill_no})
        }
    }
    ```
1. 增加FieldColor组件
    ```js
    class FieldColor extends AbstractField {
        static supportedFieldType = ['integer'];
        static template = 'OWLFieldColorPills';
        static components = { ColorPill };

        ...
    }
    fieldRegistry.add('int_color', FieldColor)
    ```
1. 在FieldColor组件中增加方法
    ```js
    constructor(...args) {
        super(...args);
        this.totalColors = Array.from({length:10}, (_,i) => (i+1).toString());
    }
    async willStart() {
        this.colorGroupData = {};
        var colorData = await this.rpc({
            model:this.model, 
            method: 'read_group',
            domain: [], 
            fields: ['color'],
            groupBy: ['color'],
        });
        colorData.forEach(res => {
            this.colorGroupData[res.color] = res.color_count;
        });
    }
    colorUpdated(ev) {
        this._setValue(ev.detail.val)
    }
    ```
1. 将js，scss文件注册到后端assets中



### 工作原理
#### 步骤3
* 增加了QWeb模板文件，其中有两个模板
    * 一个是给color pill的
    * 一个是给字段本身的
    * 使用两个模板的是为了展示子组件
* 模板中使用了ColorPill标签，用于实例化子组件
    * 在该标签中，我们传入了active和pill_no属性
    * 在子组件模板中，这些属性被作为props使用
    * t-on-color-updated属性用于监听子组件触发的自定义事件

##### 要点
* Odoo14同时支持控件系统和OWL框架
* 它们都使用QWeb模板，但是不同的是QWL的QWeb模板来自于老的QWeb模板
    * 所以需要增加owl="1"属性

#### 步骤6
* 导入OWL工具：AbstractField和fieldRegistry
    * AbstractField：用于字段的一个抽象QWL组件
    * fieldRegistry：用于注册字段组件

#### 步骤7
* 我们创建了ColorPill组件
* template是从外部XML文件中加载的模板名字
* ColorPill有一个pillClicked方法
    * 当用户点击颜色药丸时被调用
    * 内部会触发color-updated事件
    * 该事件会被FieldColor组件通过t-on-color-updated捕获住

#### 步骤8，9
* 通过扩展AbstractField创建了FielColor组件
* components变量用于定义模板中要使用的子组件
* colorUpdated方法会在用户点击药丸时被调用
    * 使用setValue方法来修改字段值，该值也会被保存进数据库