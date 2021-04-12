

# 第十四章_CMS网站开发

[TOC]




## 管理静态资源

### 总述
* 现代网站都包含有大量的JS和CSS文件
* 当浏览器加载页面时，这些静态文件需要独立向服务器请求
    * 这会导致请求数变多，网站速度变慢
* 为了避免上述问题，大部分网站都使用**静态资源**来将多个文件合并


### 资源包（asset bundle）
#### 原因
* 在Odoo中，管理静态资源并不像其他应用一样简单
    * Odoo有大量的不同应用和代码基
    * 不同的应用基于不同的目的构建，使用不同的界面
    * 应用间不会共享公共的代码
* 所以在部分场景下需要加载相同的资源，但是我们又不想在所有场景下都做这个工作
    * 在一个页面中加载不需要的今天资源不是一种好的实践
* 为了避免在所有应用中都加载额外的资源，Odoo引入了**资源包**的概念
    * 资源包的作用就是将所有的JS和CSS文件合并成一个文件，并最小化其文件大小
    * Odoo代码基有资源包，不同的代码基也有不同的资源包

#### 资源包类型
##### web.assets_common
* 该资源包包含了所有的对于所有应用通用的基础工具集资源
    * 像：jQuery，Underscore.js，字体文件
* 该资源包被用于前端，后端、POS、报表，等等模块中
* 这些通用资源被加载于Odoo中任何地方
* 该资源包也包含了boot.js文件，该文件被用于Odoo模块系统

##### web.assets_backend
* 该资源包被用于Odoo的后端（ERP部分）
* 该资源包包含了所有与网页客户端、视图、控件、动作管理器相关的代码

##### web.assets_frontend/website_assets_frontend
* 该资源包被用于Odoo的前端（网站部分）
* 该资源包包含了所有与网站应用相关的代码
    * 像：电子商务、博客、在线通知、论坛、实时聊天
* 该资源包并没有包括与网站编辑器和网站构建器相关的代码

##### web.assets_editor/web_editor.summernote
* 该资源包包含了所有与网站编辑器和网站构建器相关的代码
* 只有用户有编辑权限，这些资源包才会加载到网页上

##### web.report_assets_common
* QWeb报只是从HTML中生成一个PDF文件
* 该资源包在报表布局中才会被加载


#### 重点
* 特定应用的资源包
    * point_of_sale.assets
    * survey.survey_assets
    * mass_mailing.layout 
    * websit_slides.slide_embed_assets
* Odoo使用AssetBundle类来管理今天资源
    * 位置：/odoo/addons/base/models/assetsbundle.py
* AssetBundle功能
    * 合并多个JS和CSS文件
    * 最小化JS和CSS文件
    * 内建支持CSS预处理器



### 自定义资源
#### 总述
* 如前所述，Odoo中不同代码基有不同的资源
* 你需要为自定义的JS和CSS文件选择正确的资源包，才能获得正确的结果
    * 例子：如果你需要设计一个网站，你就需要将文件放入web.assets_frontend中
* 在极少数情况下，你需要创建一个全新的资源包

#### 创建资源包步骤
1. 创建QWeb模板，并加入JS、CSS、SCSS文件
    ```xml
    <template id="my_custom_assets" name="My Custom Assets">
        <link rel="stylesheet" type="text/css" href="xxxxx"> 
        <link rel="stylesheet" type="text/css" href="xxxxx"> 
        <link rel="stylesheet" type="text/JavaScript" src="xxxxx"> 
    </template>>
    ```
1. 在需要加入资源包的QWeb模板中使用t-call-assets
    ```xml
    <template id="some_page">

        <head>
            <t t-call-assets="my_module.my_custom_assets" t-js="false"/>
            <t t-call-assets="my_module.my_custom_assets" t-css="false"/>
        </head>
    </template>
    ```

#### 工作原理
##### 步骤1
* 我们创建了外部ID为my_custom_assets的QWeb模板
* 在该模板中，你需要列出所有的CSS,SCSS,JS文件
* Odoo首先会编译SCSS文件为CSS文件
* Odoo然后会将所有的CSS和JS文件分别合并成独立的CSS和JS文件

##### 步骤2
* 我们在模板中加载了CSS和JS资源
* t-css,t-js属性被用于加入样式表和脚本


#### 重点
* 在大部分网站开发中，你只需要将JS、CSS文件加入已存在的资源包即可
* 添加一个新资源包的情况非常少见
    * 只会在你不使用Odoo CMS功能，开发一个网页或者应用时才会遇到


### 更多
* 在Odoo中调试JS代码是非常难的
    * AssetBundle会合并所有的JS文件为一个文件，并且最小化它们
    * 通过使能带资源的开发者模式，你可以忽略资源包
    * 这时网页将会独立的加载静态资源以方便调试
* 合并的资源一旦生成就会保存到ir.attachment模型中
    * 这之后，资源和从附件中获取
    * 如果要重新生成资源，需要今天调试菜单，选择 Regenerate Assets Bundles
* Odoo只会生成一个资源
    * 在开发过程中，这个行为比较头痛，这会导致频繁的重启服务器
    * 为了克服这个问题，你可以在启动命令行中使用dev=xml参数，这个让服务器直接加载资源




## 为网页加入CSS和JS
* 以下操作会覆盖掉主网站模板，注入自定义的代码

### 操作步骤
1. 添加views/templates.xml文件，并加入一个视图
    ```xml
    <odoo>
        <template id="assets_frontend" inherit_id="web.assets.frontend">
            <xpath expr="." position="inside">
                <!-- 下面的步骤会加入代码 -->
            </xpath> 
        </template>>
    </odoo>
    ```
1. 加入CSS和SCSS文件
    ```xml
    <link rel="stylesheet" type="text/css" href="xxxxx.css"/>>
    <link rel="stylesheet" type="text/scss" href="xxxxx.scss"/>>
    ```
1. 加入JS文件
    ```xml
    <script type="text/javascript" src="xxxx.js"/>>
    ```
1. 在xxx.css中加入代码
    ```css
    body main {
        background: #b9ced8;
    }
    ```
1. 在xxx.scss中加入代码
    ```scss
    $my-bg-color: #1C2529; 
    $my-text-color: #D3F4FF; 
    nav.navbar {
        background-color: $my-bg-color !important;
        .navbar-nav .nav-link span {
            color: darken($my-text-color, 15);
            font-weight: 600;
        }
        footer.o_footer {
            background-color: $my-bg-color !important;
            color: $my-text-color;
        }
    }
    ```
1. 在xxx.js中加入代码
    ```js
    odoo.define('my_library', function(require) {
        var core = require('web.core')
        alert(core._t('Hello World));
        return { 
            //加入需要导出的函数
        }
    });
    ```




### 工作原理
* 在Odoo CMS的基础上，存在着一个被称为QWeb的XML模板引擎
* 资源包也是使用QWeb模板进行创建的
* 上面的步骤中，我们扩展了web.assets_frontend

#### 要点
##### CSS/SCSS文件顺序
* 对于CSS/SCSS文件，有时候顺序比较重要
* 如果你希望覆盖掉其他插件的样式，就需要保证你的文件在原始文件加载后再加载
* 可以通过调用视图的优先级字段来实现或者直接继承插件的视图

##### SCSS支持
* Odoo内建了对SCSS预处理的支持
* Odoo会自动将SCSS编译成CSS

##### JS脚本
* 为了避免出现JS的排序问题，Odoo使用了类似于RequireJS的算法
* 代码中使用了odoo.define()需要两个参数，定义的命名空间、包括具体实现的函数

###### 命名空间
* 如果你想使用JS扩展来开复杂的产品，你可以将代码分成不同逻辑组件，将其定义在不同的函数中
* 你可以通过用require导入这些函数，来重用它们
* 定义命名空间是可以增加模块名，使用点号进行分隔，可以避免后面的命名空间冲突

###### 定义函数
* 该函数只有一个参数require
    * 该参数是一个函数，用于获取其他命名空间的引用
    * 使用这个可以和odoo进行完全的交互，而不用使用全局的odoo对象
* 该函数可以返回一个包含引用的对象
    * 如果你返回了一些引用，你就可以在其他函数中使用它们
* 例子
    ```js
    odoo.define('my_module', function(require) {
        var test = { 
            key1: 'value1', 
            key2: 'value2',
        };
        var square = function(number) {
            return 2 * 2;
        };
        return { 
            test: test,
            square: square,
        }
    });
    ```
    其他文件
    ```js
    odoo.define('another_module', function(require) {
        var my_module = require('my_module');
        console.log((my_module.test.key1));
        console.long('square of 5 is',my_module.square(5))
    })
    ```


### 更多
* 为了提高性能，Odoo给前端只加载最小量的JS脚本
* 所有其他的资源包中的脚本都会按需进行延时加载






## 创建/修改QWeb模板

### 如何操作
1. 在controllers/main.py中增加控制器
    ```python
    class Main(http.Controller):
        @http.route('/books', type='http', auth='user', website=True)
        def library_books(self) :
            return request.render(
                'my_library.bools', {
                    'books': request.env['library.book'].search([])
                })
    ```
1. 在views/templates.xml中增加最小化模板
    ```xml
    <odoo>
        <template id="books">
           <t t-call="website.layout"> 
                <!-- 增加页面元素 -->
           </t>
        </template>>
    </odoo>
    ```
1. 在website.layout中，增加页面内容
    ```xml
    <div class="oe_structure">
        ...
    </div>
    <div class="container">
        <t t-foreach="books" t-as="book">
            ... 
            <h3 t-field="book.name"/>
            <t t-if="book.date_release">
                <div t-field="book.date_release" class="text-muted"/>
            </t>

            <ul>
                <li t-foreach="book.author_ids" t-as="author">
                    ... 
                </li>
            </ul>
        </t>
    </div>
    <section class="container" contenteditable="False">
        ...
    </section>
    ```


### 工作原理
#### 步骤1
* 向控制器传入了自定义值
* 该自定义值将会从控制器传入QWeb模板中

#### 步骤2，3
* 创建了一个名叫books的模板，用于生成必须的HTML代码来以列表方式展示书
* 所有包含在带t-call属性的t元素中的代码，都会使Odoo用website.layout模块来渲染页面并将我们的内容插入模板
* website.layout模板包含了所有的工具：Bootstrap,字体
* website.layout也包含了默认的页面头，页面尾，摘要，编辑功能
* 使用这种方式，我们可以在不重复输入相同代码的情况下，拥有一个完全的带菜单、页尾和页面编辑的网页
    * 如果不是用t-call="website.layout"，将不会得到默认值的页头，页尾和网页编辑功能


### 循环
* 为了使用记录集或可迭代的数据类型，我们需要通过列表来构建一个循环
* 在QWeb模板中，我们可以使用t-foreach来完成该功能
* 迭代可以在t元素中进行
* 例子
    ```xml
    <t t-foreach="[1,2,3,4,5]" t-as="num">
       <p><t t-esc="num"/></p> 
    </t>
    ```
    渲染后结果
    ```xml
    <p>1</p>
    <p>2</p>
    <p>3</p>
    <p>4</p>
    <p>5</p>
    ```
* 你可以在任意元素中写入t-foreach和t-as属性，此时该元及其内容将会在可迭代对象的每个项中重复出现
* 在我们的例子中国，看一下t-call元素内部真正生成内容的地方
* 该模板会带着一个上下文件被进行渲染
    * 在t-foreach元素中有一个名叫books的集合变量，可以通过它进行迭代遍历
    * t-as属性是强制的，被用于作为遍历变量的名字，来存取迭代数据
* 这个结构最常用的方式就是遍历记录集，可以用于任何可遍历的Python对象

#### 额外变量
* 在t-foreach循环中，你可以访问一对从t-as属性值中派生出来的额外变量
* 在前面示例中就是指book, 我们可以访问book_odd变量，
    * 值为true，表示遍历到了奇数个对象
    * 值为false, 表示遍历到了偶数个对象
* book_index: 返回遍历中的当前索引值（从0开始）
* book_first: 如果为true,表示当前元素是第一个
* book_last: 如果为true,表示当前元素是最后一个
* book_value: 如果遍历的对象是一个字典，该对象包含了所有项的值
    * 例子中，book是通过字典键来进行遍历的
* book_size: 集合的大小
* book_even: 与book_odd类似，基于遍历索引
* book_parity: 对于偶数项包含even，对于奇数项包含odd


### 动态属性
* QWeb模板可以动态的设置属性的值，可以通过下面三种方式实现

#### 方式1
* 通过t-att-$attr_name实现
* 在模板渲染的时候，$attr_name属性会被创建，其值可以是任何的有效Python表达式
* 表达式会在当前上下文中进行计算，并将结果设置为该属性的值
* 例子：
    ```xml
    <div t-att-total="10 + 5 + 5"/>
    ```
    渲染成
    ```xml
    <div total="20"/>
    ```

#### 方式2
* 通过t-attf-$attr_name实现
* 该方式类似于方式1，不同之处在于只有在{{...}}和#{...}中的字符串才会被求值
* 该特点对于值混合在字符串中非常有用
* 该方式经常被用于class求值
* 例子：
    ```xml
    <t t-foreach="['info', 'danger', 'warning']" t-as="color">
        <div t-attf-class="alert alert-#{color}">
            xxxx
        </div>
    </t>
    ```
    渲染成
    ```xml
    <div class="alert alert-info">
        xxxx
    </div>
    <div class="alert alert-danger">
        xxxx
    </div>
    <div class="alert alert-warning">
        xxxx
    </div>
    ```

#### 方式3
* 通过t-att=mapping选项
* 该选项接收一个字典，之后模型会渲染该字典中的数据，将其转换为属性名和属性值
* 例子：
    ```xml
    <div t-att="{'id':'my_el_id', 'class':'alert alert-danger'}"/>
    ```
    渲染成
    ```xml
    <div id="my_el_id" class="alert alert-danger"/>
    ```



### 字段
#### 总述
* h3和div标签中使用了t-field属性
* 该属性的值必须是一个长度为1的记录集
* 该属性允许用户在以编辑模式打开网页时，修改网页的内容
    * 当你保存网页时，更新后的值会被保存进数据库
    * 该操作需要进行权限检查，只有在当前用户对已显示记录有写权限时才会执行
* 使用t-optionis属性你可以给字段渲染器传入一个字典选项，包括使用的控件
    * 目前对于后端没有大量的控件可以使用，所有这里的选择比较少

#### 例子
对二进制属性，在界面上显示一张图片
```xml
<span t-field="author.image_small" t-options="{'widget':'image'}"/>
```

#### 限制
* 只能用于记录集
* 无法用于t元素中，必须使用HTML元素，像span、div


#### 与t-esc对比
* t-esc属性没有记录集的限制，可以使用任何数据类型
* t-esc是不可编辑的
* t-esc显示原始数据，t-field会基于用户语言来显示数据



### 条件
#### 总述
* 注意到分隔显示的发布日期是被包裹在带t-if属性的t元素中的
* t-if属性值会被作为Python代码来求值
* 包含t-if的元素只有在t-if属性值是真值的时候才会被渲染

#### 例子
```xml
<div t-if="state == 'new'">
    xxxxx
</div>
<div t-elif="state == 'progress'">
    xxxxx
</div>
<div t-else="">
    xxxx
</div>
```


### 设置变量
* QWeb模板运行直接在模板中定义变量
* 在定义变量后，你可以在后面的模板中使用该变量
* 例子
    ```xml
    <t t-set="my_var" t-value="5 + 1"/>

    <t t-esc="my_var">
    ```


### 子模板
* 如果需要开发大型应用，管理巨大的模板文件是很困难的
* QWeb模板支持子模板功能，可以将巨大模板文件分割成更小的子模板文件，可以在其他模板中重用它们
* 可以使用t-call属性来使用子模板
* 例子
    ```xml
    <template id="first_template">
        <div> xxxx </div> 
    </template>

    <template id="second_template">
        <t t-call="first_template">
    </template>
    ```



### 行内编辑
* 用户可以在编辑模式下的网页中直接修改记录
* 由t-field节点加载的数据默认就是可编辑的
* 如果用户修改了这种节点的值并保存了网页，该值会被更新到后端
    * 不要担心，为了更新记录，用户需要对该记录有写权限
* 注意到t-field只能用于记录集数据，为了显示其他类型数据，你可以使用t-esc
    * 该元素的工作方式类似于t-field，但是唯一的区别就是t-esc是不可编辑的、可以用于任何类型的数据
* 如果你想关闭网页编辑功能，可以使用contenteditable=False属性
    * 该属性可以使元素成为只读




### 更多
#### 修改模板
* 要修改已存在的模板，你可以像视图继承的那样，使用inherit_id属性和xpath元素
* 例子
    ```xml
    <template id="books_ids_inh" inherit_id="my_library.books">
        <xpath expr="xxxx" position="replace">
            <b>Authors(<t t-esc="len(book.author_ids)"/>)</b>
        </xpath> 
    </template>
    ```

#### 原理
* 继承工作完全像视图，是因为在内部QWeb模板是一种qweb类型的常规视图
* template元素是record元素加上一些属性的简化写法
* 没有理由不使用template的便利性，但是你需要知道实现原理
    * template元素在ir.ui.view模型中创建了一条qweb类型的记录
    * 依靠于template的name和inherit_id属性，视图记录中的inherit_id字段会被设置









## 管理动态路由

### 如何操作
1. 在main.py中为书详情增加路由
    ```python
    @http.route('/books/<model("library.book"):book>', type='http', auth='user', website=True)
    def library_book_detail(self, book) :
        return request.render('my_library.book_detail', { 
            'book': book
        })
    ``` 
1. 在templates.xml中增加书详情模板
    ```xml
    <template id="book_detail" name="Books Detail">
       <t t-call="website.layout">
            ...
            <div t-field="book.html_description">
       </t> 
    </template>
    ```
1. 在数列表模板中增加按键，重定向到书详情页面
    ```xml
    <a t-attf-href="/books/#{book.id}" class="xxx">
        <i class="fa fa-book"/>Book Detail
    </a>
    ```


### 工作原理
#### 步骤1
* 创建了书详情页面的动态路由
* /books/<model("library.book"):book> 接收带整型的URL，像/books/0
* Odoo会将整型为了library.book模型的记录ID
* 当URL被访问，Odoo会获取一个记录集，将其作为参数传入对应的函数中
* 例子：当在浏览器中输入/books/1时
    * library_book_detail函数中的book参数，将会被赋值于一个包括library.book模型ID为1的记录的记录集
    * 将book记录集传给模板my_library.book_detail，并渲染该模板

#### 步骤2
* 我们创建了一个新模板book_detail用于绘制书详情页面
* 该页面使用了简单的Bootstrap结构，我们添加了html_description字段
* html_description字段有一个field类型，所以可以在该字段中存储HTML数据
* Odoo会自动添加对HTML类型字段的代码片段拖放支持
    * 这样就可以在书详情页面使用代码片段
    * 被拖入HTML字段中的代码片段会被保存到book记录中，这样就可以为不同书设计不同内容


#### 注意
* 路由也支持domain过滤
* 例子：限制访问书名为Book 1的书
    ```python
    /books/<model("library.book", "[('name','!=','Book 1')]"):team>/submit    
    ```


### 更多
* Odoo使用werkzeug库来处理HTTP请求，只增加了一个薄的包装层来更容易处理路由
* 像<model("library.book"):book>路由就是Odoo自己实现的
* 也支持werkzeug路由的所有特征
    * /page/\<int:page\>  接收整型
    * /page/\<any(about,help):page_name\> 只接收固定的值
    * /pages/\<page\> 接收字符串
    * /pages/\<category\>/\<int:page\> 接收多个值




## 给用户提供静态代码片段

### 总述
* Odoo的网页编辑器提供了许多编辑构建块
    * 可以被拖动到页面上，并根据需要被编辑
* 构建块总体上分成两类：静态的、动态的
    * 静态代码片段是固定的，只有用户可以改变它
    * 动态代码片段依赖于数据库的记录，会根据记录值而改变


### 如何操作
1. 增加views/snippets.xml文件
    ```xml
    <template id="snippet_book_cover" name="Book Cover">
        <section>
        .... 
        </section>
    </template>>
    ``` 
1. 将模板加入代码片段列表中
    ```xml
    <template id="book_snippets_options" inherit_id="website.snippets">
         <xpath expr="xxxx">
            <t t-snippet="my_library.snippet_book_cover"/>
            <t t-thumbnail="xxxxxx"/>
         </xpath>
    </template>>
    ```
1. 将图片加入/my_library/static/src/img目录



### 工作原理
* 静态代码片段本质上就是一段HTML代码

#### 步骤1
* 我们使用了HTML来创建一个QWeb模板，在这个HTML中我们使用了Bootstrap列结构，但是你可以使用任何HTML代码
* 由于代码片段中的HTML代码会随着用户的拖放动作被加入到页面中
* 所有总的来说，使用section元素和Bootstrap类是一个很好的选择
    * 这样Odoo的编辑器可以提供开箱即用的编辑、背景色选择和缩放控制功能


#### 步骤2
* 在片段列表中注册我们的代码片段，你需要继承website.snippets来注册一个片段
* 在网页编辑器界面，片段是基于他们的用途分成不同的区域的
    

#### 注意
* website.snippets模板包括了所有的默认片段
    * 可以在/addons/website/views/snippets/snippets.xml中查询到
* 当片段使用了正确地Bootstrap结构后，Odoo会加入一些默认的选项
    * 例子中，我们可以给片段设置背景色、背景图、宽、高 等等


### 更多
* 在这种情况下，静态片段就不需要额外的JS代码了
    * Odoo提供了许多开箱即用的选项和控制，这些对于静态片段来说是足够用了。
    * 在website/views/snippets.xml中找到所有的选项
* 片段选择也支持data-exclude、data-drop-near、data-drop-in属性
    * 这些属性决定了当片段被拖出片段栏时，可以被放置在什么地方





## 从用户获取输入

### 总述
* 在网页开发中，经常需要创建表单来从用户那里获取输入
* 本节中，我们会给页面创建一个HTML表单，来让用户上报相关书的问题


### 准备
1. 给library.book增加字段，新建book.issues模型
    ```python
    class LibraryBook(models.Model) :
        book_issue_id = fields.One2many('book.issue', 'book_id')

    class LibraryBookIssuses(models.Model) :
        _name = 'book.issue'

        book_id = fields.Many2one('library.book', required=True)
        submitted_by = fields.Many2one('res.users')
        issue_description = fields.Text()
    ```
1. 在书界面增加book_issues_id字段
    ```xml
    <group string="Book Issues">
        <field name="book_issue_id" nolable="1">
            <tree>
                <field name="create_date"/>
                <field name="submitted_by"/>
                <field name="issue_description"/>
            </tree>
        </field>
    </group>
    ```
1. 给book.issue模型增加访问权限



### 如何操作
1. 在main.py中增加新路由
    ```python
    @http.route('/books/submit_issues', type='http', auth='user', website=True)
    def books_issues(self, **post) : 
        if post.get('book_id') :
            book_id = int(post.get('book_id))
            issue_description = page.get('issue_description')
            request.env['book.issue'].sudo().create({
                'book_id': book_id,
                'issue_description': issue_description,
                'submitted_by': request.env.user.id
            })
            return request.redirect('/books/submit_issues?submitted=1')
        return request.render('my_library.books_issue_form', {
            'books': request.env['library.book'].search([]),
            'submitted': post.get('submitted', False)
        })
    ```
1. 增加一个带HTML表单的模板
    ```xml
    <template id="books_issue_form" name="Book Issues Form">
        <t t-call="website.layout">
            <div class="xxxx">
                ....
            </div>
        </t>
    </template>
    ```
1. 增加一个条件展示的头
    ```xml
    <t t-if="submitted">
        <h3 class="xxx">
            <i class="xxx"/>
            Book Submitted Successfully
        </h3>
        <h1> Report the another book issue </h1>
    </t>
    <t t-else="">
        <h1> Report the book issue </h1>
    </t>
    ```
1. 增加表单
    ```xml
    <form method="post">
        <input type="hidden" name="csrf_token" t-att-value="request.csrf_token()"/>
        <div class="form-group">
            <label> Select Book</label>
            <select name="book_id">
                <t t-foreach="books" t-as="book">
                    <option t-att-value="book.id">
                        <t t-esc="book.name">
                    </option>
                </t>
            </select>
        </div>
        <div class="form-group">
            <label> Issue Description </label>
            <textarea name="issue_description" />
        </div>
        <button type="submit">Submit</button>
    </form>
    ```


### 工作原理
#### 步骤1
* 创建了一个路由来提交书问题
* 函数中**post参数接收URL中的所有查询参数，可以从中获取到已提交的表单数据
* 例子中，我们使用相同的控制器来显示页面和提交问题
    * 如果在post参数中发现了数据，就创建一个book.issue模型的新记录并使用submitted查询参数重定向到当前页面
    * 这样用户就可以看见问题提交后的一个反馈信息并且如果愿意可以继续提交另一个问题

##### 注意
* 由于一个普通用户没有创建新问题记录的权限，所有我们需要使用sudo()来创建记录
* 虽然用户是从网页上提交了一个问题，但是还是有必要创建一个问题记录的
* 该例子也是sudo()的一个实际应用


#### 步骤4
* 我们给form增加了三个字段：csrf_token，要选择的书、问题描述
* 最后两个字段是要从网页上获取的输入
* csrf_token是用来避免CSRF攻击的


##### 提示
* 在某些情况下，如果你想禁止csrf校验，可以在路由上使用crsf=False
    * 例子：@http.route('/url', type='http', auth='user', website=True, csrf=False)



### 更多
* 你可以将路由页面分开，对于提交的数据，可以在表单上增加action属性  
    ```xml
    <form action="/my_url" method="post">
        ...
    </form>
    ```
* 可以通过在路由上增加method参数来限制get请求
    ```python
    @http.route('/my_url', type='http', method='POST', auth='user', website=True)
    ```