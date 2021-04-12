

# 参考_JS框架


[TOC]


## 概述

### 设计目标
* web客户端：是一个私有的web应用，用于查看和编辑业务数据
    * 是一个单页应用（页面不会重载，只有需要时从服务器获取新数据）
* 网站：是Odoo的公开部分
    * 允许未识别用户浏览部分内容、购物、执行大量动作
    * 这是一个典型的网站：带有控制器的各种路由，一些可以工作的js脚本
* POS: 是POS模块的界面，是一个定制过的单页应用



## web客户端

### 单页应用
* 总之，webClient，WebClient类的实例，是整个用户界面的根组件
    * 作用是组织各种不同的子组件
    * 提供服务：RPC，本地存储。。。 
* web客户端作为一个单页应用来运行
    * 当用户执行一个动作时，并不需要从服务器请求全部的页面，而只需要请求需要的部分，替换更新视图即可
    * 它也管理url，使客户端状态保持同步
* 当用户在Odoo上操作时，客户端会创建和销毁大量的子组件
    * 状态是高度动态化的，任何时间每个控件都可以被销毁


### JS代码概述
客户端代码位于web/static/src/js目录中
* boot.js : 用于定义模块系统，需要第一个被加载
* core/ : 存放一些低层次的构建块代码
    * 包括：类系统、控件系统、并发工具
* chrome/ : 存放大型的控件
    * abstract_web_client.js、chrome/web_client.js ： 定义了WebClient控件，是客户端的根控件
    * action_manager.js : 用于转换动作给控件
    * search_X.js ： 定义了搜索视图（从服务器的角度）
* fields/ : 存放字段控件
* views/  : 存放视图



## 资源管理
* 在Odoo中管理资源并不像其他应用那么直接
    * 原因之一：我们会遇到很多种情况，只需要部分资源
* 例子
    * web客户端、POS界面、网站、移动应用它们所需要的资源都是不一样的
    * 有一些资源很大但是又很少使用
    * 有些情况下我们需要延时加载资源
* 一个主要想法就是使用xml来定义一组包(bundle)
    * 一个bundle被定义成一组文件集合(js，css，scss)
    * Odoo中最重要的bundle定义在addons/web/views/webclient_templates.xml中
    ```xml
    <template id="web.assets_common" name="Common Assets (used in backend interface and website)">
        <link rel="stylesheet" type="text/css" href="/web/static/lib/jquery.ui/jquery-ui.css"/>
        ...
        <script type="text/javascript" src="/web/static/src/js/boot.js"></script>
        ...
    </template>
    ```
    * 该bundle可以使用t-call-assets插入到模板中
    ```xml
    <t t-call-assets="web.assets_common" t-js="false"/>
    <t t-call-assets="web.assets_common" t-css="false"/>
    ```
* 在服务器渲染模板时，t-call-assets指令会进行以下操作
    * 所有的scss文件会编译成css文件，名字为 file.scss.css
    * 在debug=assets模式下
        * t-js为false的指令会被替换成一组css文件的样式表标签
        * t-css为false的指令会被替换成一组js文件的脚本标签
    * 不在debug=assets模式下
        * css文件会被合并进行最小化，生成一个样式表标签
        * js文件会被合并进行最小化，生成一个脚本标签


### 主bundle
* 当服务器启动时，odoo会检查bundle中每个文件的时间戳，如果有必要会创建/重建对应的bundle
* 开发时需要知道的bundle
    * web.assets_common: 包含了web客户端、网站、POS界面都需要的公共资源，包括框架的低层次构建块，boot.js文件
    * web.assets_backend: 包含了针对web客户端的资源，包括 webClient、动作管理器、视图
    * web.assets_frontend: 包含了针对公共网站的资源，包括电子商务、论坛、博客、事件管理器


### 向bundle中添加文件
* 在addons/web中向一个bundle添加文件的方法非常简单
    * 在webclient_templates.xml中添加一个脚本、样式表标签
* 当工作在其他插件中时，我们需要从该插件中添加文件。这时需要三个步骤
    1. 在views/目录中添加assets.xml文件
    1. 在manifest的data键中，添加views/assets.xml
    1. 对目标bundle创建一个视图继承，使用xpath来添加文件
    ```xml
    <template id="assets_backend" name="helpdesk assets" inherit_id="web.assets_backend">
        <xpath expr="//script[last()]" position="after">
            <link rel="stylesheet" type="text/scss" href="/helpdesk/static/src/scss/helpdesk.scss"/>
            <script type="text/javascript" src="/helpdesk/static/src/js/helpdesk_dashboard.js"></script>
        </xpath>
    </template>
    ```

### 当文件不加载/不更新时如何做
一个文件不能被正确的加载有很多原因，这里有一些方法可以解决这个问题
* 当服务器启动后，就无法知道一个资源文件是否被修改过。所以你可以重启服务器来重生成资源
* 检查控制台(在浏览器开发工具中)，保证没有明显的错误信息
* 尝试在文件开始出添加console.log，这样就是看成文件是否被加载
* 在用户界面的调试模式下，有一个选项可以强制服务器更新资源文件
* 使用debug=assets模式，该模式实际上忽略的资源bundle
* 最后对于开发者来说，最方便的方式就是启动服务器时增加 --devl=all 选项
    * 该选项会激活文件观察者功能，会自动在需要时无效掉资源
* 记住刷新你的页面
* 记住保存你的代码文件




## JS模块系统

### 概述
* 一旦能够在浏览器中加载js文件后，就需要保证以正确的顺序来加载它们
* 为了做到这一点，Odoo定义了一个小的模块系统
* Odoo的模块系统灵感来源于AMD，通过全局变量odoo中的define方法实现。我们通过调用该函数来定义每个js模块
* Odoo框架中，模块就是一段需要尽快执行地代码
    * 还有一个名字和潜在的依赖
    * 当依赖被加载后，模块也就被加载完成了
* 模块值就是定义该模块的函数的返回值
* 如果依赖缺失或者未准备好，模块会加载失败，并在控制台中输出警告信息
* 循环依赖是不支持的

#### 例子
```js
// in file a.js
odoo.define('module.A', function (require) {
    "use strict";

    var A = ...;

    return A;
});

// in file b.js
odoo.define('module.B', function (require) {
    "use strict";

    var A = require('module.A');

    var B = ...; // something that involves A

    return B;
});
```
另一种方式
```js
odoo.define('module.Something', ['module.A', 'module.B'], function (require) {
    "use strict";

    var A = require('module.A');
    var B = require('module.B');

    // some code
});
```

### 定义模块
#### odoo.define()参数
* moduleName : js模块的名字，必须是唯一字符串
    * 一般情况下要包含插件的名字
    * 例子：web.Widget 描述了web插件中的一个模块，导出为一个Widget类
* dependencies : 该参数可选，格式为字符串列表，每个项对应一个js模块
    * 在模块执行前，该参数描述的依赖必须要被加载
    * 如果该参数没有指定依赖，模块系统会通过toString()从函数中提取出依赖，使用正则表达式查找所有的require语句
* function : 一个函数用于定义模块，该函数返回值就是模块的返回值。该值传递给请求该模块的其他模块


#### 错误信息
* Missing dependencies : 该模块没有出现在页面中
    * 可能原因：js文件不在该页面中、模块名错误
* Failed modules : 发现js错误
* Rejected modules : 模块返回一个被拒绝的Promise。该模块（其依赖模块）没有被加载过
* Rejected linked modules : 依赖的模块是一个被拒绝的模块
* Non loaded modules : 依赖的模块是一个丢失、加载失败的模块


### 异步模块
* 该功能用于模块需要在准备好前执行一些操作，像执行一个rpc来获取数据
* 这种情况下模块可以简单的返回一个Promise，这时模块系统会等待这个Promise完成，再注册这个模块

#### 示例
```js
odoo.define('module.Something', function (require) {
    "use strict";

    var ajax = require('web.ajax');

    return ajax.rpc(...).then(function (result) {
        // some code here
        return something;
    });
});
```

### 最佳实践
* 记住模块名的约定格式： 插件名 + 模块名
* 在模块开始处定义所有的依赖，按字母顺序进行排序
* 在模块结束出定义所有要导出的值
* 避免从一个模块中导出太多的值，最好是一个模块导出一个值
* 异步模块可以简单的用于一些场景
    * web.dom_ready会返回一个Promise，来等待DOM实际准备好
    * 如果其他模块需要等待DOM准备好，可以简单的导入该模块 require('web.dom_ready')
    * 模块的代码就会在DOM准备好后才会执行
* 避免在一个文件中定义多于一个的模块
    * 定义多个模块短期内看起来很方便，但是实际上很难维护



## 类系统

### 概述
* Odoo开发预ES6发布前，在ES5中定义类的标准方法是定义一个方法，并在其原型上增加方法
    * 这种方法也不错，但是在进行继承和混入会变得比较复杂
* 所以Odoo开发了自己的类系统，受John Resig启发。
    * 基类位于class.js文件中的web.Class类



### 创建子类
* 创建子类主要使用extend()方法
* 示例
    ```js
    var Class = require('web.Class');

    var Animal = Class.extend({
        init: function () {
            this.x = 0;
            this.hunger = 0;
        },
        move: function () {
            this.x = this.x + 1;
            this.hunger = this.hunger + 1;
        },
        eat: function () {
            this.hunger = 0;
        },
    });
    ```
* 在示例中,init函数是构造函数，当类的实例被创建出来后会被调用
* 可以使用new关键字来创建类的实例


### 继承
* 继承一个已存在类比较方便，可以简单的使用超类中的extend()方法
* 当一个方法被调用时，框架会对当前调用的方法，秘密的重绑定一个特殊方法 **_super**
    * 当我们需要调用父方法时，可以使用 **_super** 来完成
* 示例
    ```js
    var Animal = require('web.Animal');

    var Dog = Animal.extend({
        move: function () {
            this.bark();
            this._super.apply(this, arguments);
        },
        bark: function () {
            console.log('woof');
        },
    });

    var dog = new Dog();
    dog.move()
    ```


### 混入
* odoo的类系统并不支持多重继承，但是当我我们需要共享一些行为时，我们可以使用混入系统
* extend()方法可以接收任务数量的参数，将它们合并到新类中
* 示例
    ```js
    var Animal = require('web.Animal');
    var DanceMixin = {
        dance: function () {
            console.log('dancing...');
        },
    };

    var Hamster = Animal.extend(DanceMixin, {
        sleep: function () {
            console.log('sleeping');
        },
    });
    ```


### 修改已存在类
* 虽然不是很常见，但是有时候我们需要修改其他类
* 目标是有算法可以修改一个类和该类的所有的（未来的和现在的）实例
* 我们可以使用include()方法来实现
* 示例
    ```js
    var Hamster = require('web.Hamster');

    Hamster.include({
        sleep: function () {
            this._super.apply(this, arguments);
            console.log('zzzz');
        },
    });
    ```
* 该方法明显是一个很危险的操作，需要尽量的小心。
* 但是odoo的构造方式，会导致有时候需要在插件中修改其他插件中一个控件/类的行为
* 注意：该方法会修改掉该类的所有实例，甚至是那些已经创建出来的实例




## 控件

### 概述
* 控件类是用户界面中一个重要的构建块,用户界面中的相当多的事件都在控件的控制下
* 控件类定义在widget.js文件中的web.Widget模块

#### 控件类特征
* 控件之间的父/子关系，使用PropertiesMixin实现
* 带安全机制的声明周期管理（例：在父控件销毁时自动销毁子控件）
* 自动渲染qweb模板
* 大量的和外部环境交互的工具函数

#### 实例
一个基本的计数控件
```js
var Widget = require('web.Widget');

var Counter = Widget.extend({
    template: 'some.template',
    events: {
        'click button': '_onClick',
    },
    init: function (parent, value) {
        this._super(parent);
        this.count = value;
    },
    _onClick: function () {
        this.count++;
        this.$('.val').text(this.count);
    },
});
```
渲染模板
```xml
<div t-name="some.template">
    <span class="val"><t t-esc="widget.count"/></span>
    <button>Increment</button>
</div>
```
使用方法
```js
// Create the instance
var counter = new Counter(this, 4);
// Render and insert into DOM
counter.appendTo(".some-div");
```


### 生命周期
 * 像很多组件系统一样，控件类有一套完善的生命周期
 * 常见的生命周期如下：调用init()，然后是willStart()，然后进行渲染，然后调用start()，最后调用destroy()

 #### Widget.init(parent)
 * 该方法是一个构建方法，用于初始化控件的基本状态
 * 该方法是同步执行的，可以被覆盖加入更多的参数
 * 参数说明
    * parent : 新控件的父控件，用于处理自动销毁和事件传递

#### Widget.willStart()
* 当控件被创建后，在加入DOM的过程中，框架会调用该方法一次
* 该方法是一个用于返回Promise的钩子，js框架会等待Promise完成，才会进入渲染步骤
* 注意：控件并不拥有DOM的根元素
* 该方法主要用于执行一些异步操作，像从服务器获取数据

#### 渲染
* 该步骤由框架自动完成
* 主要步骤
    1. 检查是否定义了模板，如果是则在渲染上下文中渲染模板并绑定到控件中
    1. 如果没有定义模板，则读取tagName键的内容，来创建对应的DOM元素
    1. 当渲染完成，将结果设置到控件的$el属性上
    1. 最后自动绑定上所有事件和自定义事件键

#### Widget.start()
* 当渲染完成后，框架会自动调用该方法
* 该方法用于执行一些渲染后的工作
* 该方法必要要返回一个Promise，来表明工作已经完成

#### Widget.destroy()
* 该方法是控件生命周期的最后阶段了。
* 当控件被销毁时，我们就需要执行所有必要的清除操作：从组件树中移除控件，解绑所有的事件...
* 当父控件被销售后，该方法会自动被调用。
    * 果控件没有父控件或者从父控件上移除，则必须要显式的调用该方法


#### 注意
* willStart和start方法并不一定会被调用
    * 控件可以被创建，然后在没有加入DOM的情况下被销毁
    * 在这种情况下，init()和destroy()会被调用，而willStart()和start()不会被调用



### 控件API
#### Widget.tagName
* 用于控件没有定义模板的情况，默认值为div
* 内容会被用来创建DOM元素并设置为控件DOM的根

#### Widget.id
* 用于生成控件DOM根元素的id属性
* 注意该属性很少会使用

#### Widget.className
* 用于生成控件DOM根元素的class属性
* 内容可以包含多个css类，像‘some-class other-class’

#### Widget.el
* 原始的控件根的DOM元素

#### Widget.$el
* jQuery包装过的el

#### Widget.template
* 用于设置QWeb模板的名字
* 如果设置了，则在控件初始化以后，启动之前，模板就会被渲染
* 模板生成的根元素会设置成控件的DOM根

#### Widget.xmlDependencies
* 用于放置一些在控件渲染之前就需要加载的xml文件，已经被加载的文件不会被重新加载
* 当你需要延迟加载模板或者在网站和web客户端之间共享控件时，可以使用该属性
* 示例
    ```js
    var EditorMenuBar = Widget.extend({
        xmlDependencies: ['/web_editor/static/src/xml/editor.xml'],
        ...
    ```

#### Widget.events
* 格式是一个 事件选择器（由空格分隔的事件名和CSS选择器）到 回调 的映射
* 回调是一个控件中方法的名字或者一个函数对象
* 无论什么情况下，this都将会被设置成控件本身
* 选择器适用于jQuery的事件委托，当DOM根的子节点匹配选择器，回调将会被触发
* 如果选择器被忽略（只有一个事件名字），事件将会被设置在控件DOM根元素上
* 示例
    ```js
    events: {
        'click p.oe_some_class a': 'some_method',
        'change input': function (e) {
            e.stopPropagation();
        }
    },
    ```

#### Widget.custom_events
* 和events类似，但是键可以是任意的字符串
* 字符串可以是由子控件触发的业务事件
* 当事件被触发后，事件会顺着控件树冒泡上来

#### Widget.isDestroyed()
* 如果控件已经被销毁了，就会返回true

#### Widget.$(selector)
* 将CSS选择器作为参数应用在控件DOM根上
* 参数
    * selector ：字符串类型，CSS选择器
* 返回值：jQuery对象
* 示例
    ```js
    this.$(selector);
    this.$el.find(selector);  //与上面的写法功能一样
    ```

#### Widget.setElement()
* 重新设置控件的DOM根
* 也用来重设DOM根的各种别名，以及取消和重设被委托的事件
* 参数
    * element : Element类型，一个DOM元素或者jQuery对象




### 将控件插入DOM
#### 方法
* Widget.appendTo(element) 渲染控件并将其作为目标元素最后一个子节点插入
* Widget.prependTo(element) 渲染控件并将其作为目标元素的第一个子节点插入
* Widget.insertAfter(element) 渲染控件并将其作为目标元素前一个兄弟节点插入
* Widget.insertBefore(element) 渲染控件并将其作为目标元素后一个兄弟节点插入

#### 说明
* 所有方法都接受任何jQuery方法接收的参数
    * 包括CSS选择器、DOM节点、jQuery对象
* 所有方法都返回一个Promise
* 所有方法都进行以下操作
    1. 使用renderElement()来渲染控件的根节点
    1. 使用匹配的jQuery方法将控件根元素插入到DOM中
    1. 启动控件，返回启动的结果



### 控件向导
#### 不要使用标识符
* 标识符（id属性）是要尽量避免使用的
* 在通用应用和模块中，id会限制组件的重用，使代码变得脆弱
* 大部分情况下，标识符可以不使用、使用类、使用DOM节点的引用、使用jQuery元素
* 如果一定需要id，可以使用_.uniqueId()来生成
    ```js
    this.id = _.uniqueId('my-widget-');
    ```

#### 不要使用公共的CSS类名
* 像“内容”和“导航”这样的类名也许匹配希望的意思和语义，但是其他开发者也会有相同的需求，会导致命名冲突和意外行为
* 通用的类名应该加上所属组件的名字作为前缀

#### 不要使用全局选择器
* 一个组件可能会在单一页面中被使用很多次（像odoo中的仪表板），查询会限制在组件的范围内
* 无过滤的选择器像会导致意外或者不正确的行为
    * $(selector)
    * document.querySelectorAll(selector)
* odoo的web控件使用属性\$el来提供DOM根，可以直接使用$()来选择节点


#### 确保等待start完成再使用$el
```js
var Widget = require('web.Widget');

var AlmostCorrectWidget = Widget.extend({
    start: function () {
        //理论上，$el已经设置好了。但是你不知道父控件会做什么操作，最好还是先调用super
        this.$el.hasClass(....) 
        return this._super.apply(arguments);
    },
});

//错误实现
var IncorrectWidget = Widget.extend({
    start: function () {
        //父控件的promise已经丢失了，没有人会等待该控件的启动
        this._super.apply(arguments); 
        this.$el.hasClass(....)
    },
});

//正确实现
var CorrectWidget = Widget.extend({
    start: function () {
        var self = this;
        return this._super.apply(arguments).then(function() {
            //该方法有效，没有Promise会丢失，代码已可控的顺序执行：先super的,然后我们的
            self.$el.hasClass(....) 
        });
    },
});
```

#### 其他
* 需要使用QWeb来渲染html页面
* 所有的交互组件都必须继承Widget()、正确地实现、使用其API、遵守生命周期




## QWeb模板引擎

### 概述
* web客户端使用QWeb模板引擎来渲染控件（除非覆写renderElement方法）
* QWeb的JS模板引擎是基于XML的，与python的实现大部分是兼容的

### 模板加载
* 当web客户端启动时，会对/web/webclient/qweb路由进行一次RPC调用
* 服务器会返回一个定义在已安装模块的数据文件中的模板列表。正确的文件在manifest中的qweb入口中
* web客户端会等待所有的模板被加载完成，再启动第一个控件

### 延时加载
* 这种机制很适合我们的需要，但是有时候我们需要延时加载一个模板
* 例子：有一个控件很少会使用，我们希望在主文件中不要加载该模板，这样可以使web客户端更轻量一点
* 可以使用控件中的xmlDependencies键
    ```js
    var Widget = require('web.Widget');

    var Counter = Widget.extend({
        template: 'some.template',
        xmlDependencies: ['/myaddon/path/to/my/file.xml'],
        ...
    });
    ```
    * Counter控件会在willStart方法中加载xmlDependencies的文件



## 事件系统

### 概述
* Odoo支持两种事件系统
    * 简单系统：允许添加监听器、触发事件
    * 复杂系统：支持事件的’冒泡‘功能
* 事件系统实现为minxins.js文件中的EventDispatcherMixin类，会混入Widget类


### 基础事件系统
* 该系统是第一个实现的，实现为简单的总线模式
* 支持以下方法
    * on ：向事件注册一个监听器
    * off : 移除监听器
    * once : 注册只会被调用一次的监听器
    * trigger : 触发事件，会导致该事件相关的监听器都被调用
* 示例
    ```js
    var Widget = require('web.Widget');
    var Counter = require('myModule.Counter');

    var MyWidget = Widget.extend({
        start: function () {
            this.counter = new Counter(this);
            this.counter.on('valuechange', this, this._onValueChange);
            var def = this.counter.appendTo(this.$el);
            return Promise.all([def, this._super.apply(this, arguments)]);
        },
        _onValueChange: function (val) {
            //使用val做一些操作
        },
    });

    //在Counter控件中，我们需要调用trigger方法
    ... this.trigger('valuechange', someValue);
    ```


### 扩展事件系统

#### 概述
* 自定义事件系统更加先进一点，模仿了DOM事件的API
* 当事件给触发时，事件会在控件树上进行冒泡，直到到达根控件
* trigger_up方法会创建一个小的OdooEvent，将其分发到组件树中（会从触发事件的组件开始冒泡）
* custom_events 等同于一个事件字典，但是是为了odoo事件

#### OdooEvent
* 该类非常简单，有三个公开的属性
    * target : 触发事件的控件
    * name : 事件名字
    * data : 载荷
* 有两个方法
    * stopPropagation()
    * is_stopped()

#### 示例
```js
var Widget = require('web.Widget');
var Counter = require('myModule.Counter');

var MyWidget = Widget.extend({
    custom_events: {
        valuechange: '_onValueChange'
    },
    start: function () {
        this.counter = new Counter(this);
        var def = this.counter.appendTo(this.$el);
        return Promise.all([def, this._super.apply(this, arguments)]);
    },
    _onValueChange: function(event) {
        //使用event.data.val做一些操作
    },
});

 //在Counter控件中，我们需要调用trigger_up方法
... this.trigger_up('valuechange', {value: someValue});
```




## 注册

### 概述
* 在Odoo生态中，从外部（其他模块）来扩展、修改基础系统的行为是一个通用需求
* 例子：在一些视图中增加一个新的控件类
    * 通常流程就是创建一个需要的组件，将其加入注册表，web客户端的其他部分就可以感知到该组件的存在


### 注册表类型
#### 字段注册表
* 该注册表包含了所有的字段控件
* 当视图需要一个字段控件时，就会来改注册表查找
* 示例
    ```js
    var fieldRegistry = require('web.field_registry');

    var FieldPad = ...;
    fieldRegistry.add('pad', FieldPad);
    ```
* 注意：每个类都必须是AbstractField的子类

#### 视图注册表
* 该注册表包含了所有的js视图
* 每个类都必须是AbstractView的子类

#### 动作注册表
* 该注册表用于跟踪所有的客户端动作
* 当需要创建一个动作时，动作管理都会来该注册表查找事件
* Odoo12中，每个类必须是AbstractAction的子类




## 控件间通信

### 父控件向子控件
* 最简单的方法，父控件直接调用子控件的方法
* 示例
    ```js
    this.someWidget.update(someInfo);
    ```

### 子控件向父控件
* 该情况下，控件的工作只是简单地通知一下运行环境有事情发生了
* 因为我们不想让控件有一个父控件的引用，最好的方法就是使用trigger_up()触发一个事件，让其在组件树中进行冒泡
* 示例
    ```js
    this.trigger_up('open_record', { record: record, id: id});

    var SomeAncestor = Widget.extend({
        custom_events: {
            'open_record': '_onOpenRecord',
        },
        _onOpenRecord: function (event) {
            var record = event.data.record;
            var id = event.data.id;
            // do something with the event.
        },
    });
    ```


### 跨组件
* 跨组件通信可以归纳为使用总线通信
* 这并不是最好的通信形式，因为这会让代码更难维护。但是这可以让组件更独立，不会相互关联
* 示例
    ```js
    // in WidgetA
    var core = require('web.core');

    var WidgetA = Widget.extend({
        ...
        start: function () {
            core.bus.on('barcode_scanned', this, this._onBarcodeScanned);
        },
    });

    // in WidgetB
    var WidgetB = Widget.extend({
        ...
        someFunction: function (barcode) {
            core.bus.trigger('barcode_scanned', barcode);
        },
    });
    ```
    * 该示例使用了web.core来导出总线，但是这并不是必须的





## 服务

### 概述
* Odoo11上引入了服务的概念，主要目的是让子组件以可控的方式来访问运行环境，这种方式允许框架有足够的控制，并且是可测试的
* 服务系统围绕三个概念来组织的：服务、服务提供者、控件
    * 系统的工作方式如下：控件触发（使用trigger_up）事件，事件冒泡到服务提供者，服务提供者请求服务执行动作


### 服务
* 一个服务是AbstractService类的实例，包含一个名字和一些方法
* 它的工作就是执行一些操作，具体操作取决于环境
* 例子：ajax服务执行rpc操作，本地存储服务会和浏览器的本地存储进行交互

#### 示例
```js
var AbstractService = require('web.AbstractService');

var AjaxService = AbstractService.extend({
    name: 'ajax',
    rpc: function (...) {
        return ...;
    },
});
```

### 服务提供者
* 为了让服务正常工作，就需要有一个服务提供者来分发自定义的事件
* 在后端（web客户端），这个工作由主web客户端实例来完成
* 实现代码在ServiceProviderMixin类


### 控件
* 控件用于向服务发送请求，只需要简单触发一个call_serice事件（通常使用helper函数来调用）
* 事件会冒泡，用于和系统的其他部分交流意图
* 在实践中，一些函数会频繁的被调用，需要一些辅助方法来使其更容易使用
    * 例子：_rpc()是一个进行RPC的辅助函数
    ```js
    var SomeWidget = Widget.extend({
        _getActivityModelViewID: function (model) {
            return this._rpc({
                model: model,
                method: 'get_activity_view_id'
            });
        },
    });
    ```


### RPC
* rpc功能由ajax服务提供，但是大部分用户只会与_rpc辅助函数交互
* 开发Odoo过程中有两个典型的用例
    * 调用python模型的一个方法
    * 通过路由，直接调用一个控制器
* 示例
    ```js
    return this._rpc({
        model: 'some.model',
        method: 'some_method',
        args: [some, args],
    });

    return this._rpc({
        route: '/some/route/',
        params: { some: kwargs},
    });
   ``` 



## 通知

### 概述
* Odoo框架有标准的方法和用户传达各种信息：通知
* 通知显示在用户界面的右上角

#### 通知类型
* 通知：用于显示一些反馈信息，例如：用户退订一个频道
* 警告：用于显示一些重要的、紧急的信息。例如：系统的错误信息

#### 其他用途
* 通知还可以在不打扰当前工作流的情况下，向用户询问问题
* 例子：通过VOIP接听一来电，可以显示一个粘性的，带有两个按键的通知


### 通知系统
Odoo中通知系统由以下组件构成
* 一个通知控件：一个简单的控件，创建出来显示一些期望的信息
* 一个通知服务：一个服务当请求(一个自定义事件)来的时候，进行创建和销毁通知
    * 注意：web客户端是一个服务提供者
* 一个客户端动作(display_notification)：允许从python触发一个通知的显示
* 两个辅助函数：don_notify, do_warn


### 显示通知
通用的显示通知的方法是ServiceMixin类中的两个方法

#### 方法
* do_notify(title, message, sticky, className)
    * title: 字符串，作为标题显示出来
    * message: 字符串，通知的内容
    * sticky: 布尔值，true表示通知会一直显示直到用户取消。否则通知会在延后一定时间后自动关闭
    * className: 字符串，自动添加到通知的CSS类名。用于改变通知的样式
* do_warn(title, message, sticky, className)
    * title: 字符串，作为标题显示出来
    * message: 字符串，通知的内容
    * sticky: 布尔值，true表示通知会一直显示直到用户取消。否则通知会在延后一定时间后自动关闭
    * className: 字符串，自动添加到通知的CSS类名。用于改变通知的样式


#### 示例
```js
// 注意，我们调用在文本上调用 _t(字符串)来使其可以正确的被翻译
this.do_notify(_t("Success"), _t("Your signature request has been sent."));

this.do_warn(_t("Error"), _t("Filter name is required."));
```

```python
// 注意，我们调用在文本上调用 _(字符串)来使其可以正确的被翻译
def show_notification(self):
    return {
        'type': 'ir.actions.client',
        'tag': 'display_notification',
        'params': {
            'title': _('Success'),
            'message': _('Your signature request has been sent.'),
            'sticky': False,
        }
    }
```



## 系统托盘

### 概述
* 系统托盘在界面菜单栏的右部分，web客户端在此显示一些控件，像消息菜单
* 当系统托盘菜单被创建出来后，它会搜索所有的已注册的控件，将它们作为子控件添加在合适的位置
* 系统托盘控件没有特定的API。它们只是一些简单的控件，像其他控件一样使用trigger_up()来与运行环境交互


### 增加系统托盘项
* 系统托盘没有注册表，恰当的方法是将控件加入SystrayMenu.items类变量
* 例子
    ```js
    var SystrayMenu = require('web.SystrayMenu');

    var MySystrayWidget = Widget.extend({
        ...
    });

    SystrayMenu.Items.push(MySystrayWidget);
    ```

### 排序
* 在加入控件之前，系统托盘菜单会使用sequence属性来将控件排序。
    * 如果控件没有该属性，系统会使用50来替换
    * sequence值越大，位置越靠右
* 例子
    ```js
    MySystrayWidget.prototype.sequence = 100;
    ```


## 翻译管理

### 概述
* 一些翻译是在服务器端完成的（基本上所有的字符串都由服务器渲染和处理）
* 但是在静态文件中的字符串也需要翻译

### 处理步骤
1. 每个可翻译的字符串都由_t()来标注（该函数在web.core模块中）
1. 这些字符串由服务器生成恰当的po文件
1. 当web客户端被加载后，会调用/web/webclient/translations，来获取一张可翻译条文的列表
1. 在运行中当_t()被调用时，该函数会在列表中查找来发现字符串对应的翻译


### 延时求值
* js中的翻译有两个重要的函数：_t()和_lt()
* 不同点就是，_lt()是延时求值的

#### 示例
```js
var core = require('web.core');

var _t = core._t;
var _lt = core._lt;

var SomeWidget = Widget.extend({
    exampleString: _lt('this should be translated'),
    ...
    someMethod: function () {
        var str = _t('some text');
        ...
    },
});
```



## 会话

### 概述
* 该模板是web客户端提供的一个特殊模块，包含了用户当前会话中的一些信息


#### 重要键
* uid ：当前用户的ID
* user_name : 用户名字
* 用户上下文，（用户ID、语言、时区）
* partner_id ： 当前用户分配的联系人ID
* db : 当前用户所使用的数据库名字


### 添加信息到会话
* 当/web路由被加载后，系统会在模板脚本标签中注入一些额外的信息
    * 这些信息可以通过ir.http模块的session_info()来读取
* 如果你想添加一些信息，可以覆写session_info()


#### 示例
注入
```js
rom odoo import models
from odoo.http import request


class IrHttp(models.AbstractModel):
    _inherit = 'ir.http'

    def session_info(self):
        result = super(IrHttp, self).session_info()
        result['some_key'] = get_some_value_from_db()
        return result
```
获取
```js
var session = require('web.session');
var myValue = session.some_key;
```


### 注意
* 该算法是被设计用来降低web客户端和服务器的通信数据量的
* 大部分数据都是可以方便计算出来的（一个慢速的session_info()调用会降低web客户端的加载速度），在初始化早期阶段就需要的数据




## 视图

### 概述
* 视图 这个词有多种意思，这个小节是关于js代码中视图的设计，不是指架构或者其他的含义
* 在2017年，Odoo使用新的架构替换了原来的视图代码，主要目标就是从模型逻辑中分离出渲染逻辑

### 结构
* 视图被分成四个部分：一个视图类、一个控制器类、一个渲染器类、一个模型类
* API描述分别在AbstractView, AbstractController, AbstractRenderer, AbstractModel类中

#### 图示
![](https://gitee.com/cc12703/figurebed/raw/master/img/20210419140205.png)

#### 视图类
* 视图类是一个工厂类，主要工作就是获取参数，然后构造出其他三个类
* 视图类的作用在于使用正确的信息来按MVC模式设置各个部分
* 通常，该类也需要处理架构字符串和抽取各个部分的必要数据
* 注意：视图类是一个普通类而不是一个控件，在工作完成后该类可以被清除

#### 渲染器类
* 该类只有一个工作，展示在DOM元素中的数据。每个视图可以使用不同的方式来渲染数据
* 该类也会监听用户行为，通知其父类（控制器类）
* 该类是MVC模式中的V

#### 模型类
* 该类的工作是获取并保存视图的状态，通常状态是一些数据库中的记录集
* 模型是**业务数据**的拥有者
* 该类是MVC模式中的M

#### 控制器类
* 该类的工作是协调渲染器和模型，也是也web客户端其他部分交互的主入口
* 例子：当用户在搜索界面修改一些东西时，控制器的updae方法会被调用，并带有正确的信息
* 该类是MVC模式中的C



## 字段控件

### 概述
* web客户端的一个好地方是编辑和创建数据的体验
* 大部分工作都由字段控件来完成，该控件可以感知字段类型，以及该值要如何展示和修改的细节


### AbstractField类
* AbstractField是所有这类控件的基类，所有的视图（表单、列表、看板）都支持它们

#### Odoo11上的特点
* 控制会在所有视图（表单、列表、看板）中被共享
* 控件不在是字段值的拥有者，控件只是用来展示数据、和系统其它部分交互
* 控件不在需要在编辑模式和只读模式进行切换
    * 当需要发生切换时，控件会被销毁然后重新渲染出来
* 字段控件也可以在视图之外使用，所有API看起来有些尴尬，但是这些控件是被设计成可以独立使用的



### 装饰符 
* 像列表视图，字段控件也支持简单的装饰符
* 装饰符用于根据记录当前状态来指定文本颜色

#### 示例
```xml
<field name="state" decoration-danger="amount &lt; 10000"/>
```

#### 有效的装饰符
* decoration-bf
* decoration-it
* decoration-danger
* decoration-info
* decoration-muted
* decoration-primary
* decoration-success
* decoration-warning

说明
* 每个decoration-X都对应一个标准的bootstap的CSS类text-X（除去text-it,text-bf）
* 装饰符的属性值是一个有效的python表达式，求值时使用记录作为求值上下文



### 非关系字段

#### 整型(FieldInteger)
* 该控件是整数类型字段的默认控件
* 支持字段类型：整型
* 选项
    * type: 输入类型，值为text、number （默认为text）
    * step: 对于number输入类型，用户每次点击增加、减少时的数量（默认为1）
    * format: 是否要被格式化（默认为true）
* 示例
    ```xml
    <field name="int_value" 
            options='{"type": "number", "step": 100, "format": false}'/>
    ```

#### 浮点数(FieldFloat)
* 该控件是浮点数类型字段的默认控件
* 支持字段类型：浮点数
* 属性
    * digits: 显示的精确度
* 选项
    * type: 输入类型，值为text、number （默认为text）
    * step: 对于number输入类型，用户每次点击增加、减少时的数量（默认为1）
    * format: 是否要被格式化（默认为true）
* 示例
    ```xml
    <field name="factor" digits="[42,5]" options='{"type": "number", "step": 0.1}'/>
    ```

### 关系字段





## 客户端动作

### 概述
* 客户端动作是一个在web客户端界面集成用的自定义控件，类似于act_window_action
* 该组件对于已存在的视图和模型都没有紧密的关联

#### 含义
* 服务器角度：ir_action模型的一条记录，带一个char类的tag字段
* web客户端角度：继承于AbstractAction类的一个控件，可以注册到动作注册表中

#### 流程
当一个菜单项绑定到一个客户端动作时
1. 点击该菜单项会直接从服务器获取一个动作定义
1. 在动作注册表中使用特定的键查找控件
1. 将控件添加到DOM的正确位置上


### 添加动作
* 客户端动作是一个用于控制菜单栏下边屏幕区域的控件
    * 如果需要，会带一个控件面板
* 添加需要两个步骤
    1. 实现一个新控件
    1. 将控件注册到动作注册表中

#### 示例
* 创建控件
    ```js
    var AbstractAction = require('web.AbstractAction');

    var ClientAction = AbstractAction.extend({
        hasControlPanel: true,
        ...
    });
    ```
* 注册控件
    ```js
    var core = require('web.core');

    core.action_registry.add('my-custom-action', ClientAction);
    ```
* 使用
    ```xml
    <record id="my_client_action" model="ir.actions.client">
        <field name="name">Some Name</field>
        <field name="tag">my-custom-action</field>
    </record>
    ```


### 使用控制面板
* 默认情况下不显示控制面板

#### 操作步骤
1. 设置hasControlPanel为true
    ```js
    var MyClientAction = AbstractAction.extend({
        hasControlPanel: true,
        loadControlPanel: true, // default: false
        ...
    });
    ```
    说明：当loadControlPanel设置成true时，客户端动作为自动的获取到搜索视图或者控制面板视图的内容
1. 在需要更新控件面板时调用updateControlPanel()
    ```js
    hasControlPanel: true,
    ...
    start: function () {
        this._renderButtons();
        this._update_control_panel();
        ...
    },
    do_show: function () {
         ...
         this._update_control_panel();
    },
    _renderButtons: function () {
        this.$buttons = $(QWeb.render('SomeTemplate.Buttons'));
        this.$buttons.on('click', ...);
    },
    _update_control_panel: function () {
        this.updateControlPanel({
            cp_content: {
               $buttons: this.$buttons,
            },
        });
    }
    ```