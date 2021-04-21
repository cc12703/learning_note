

# 参考_QWeb

[TOC]


## 概述
* QWeb是Odoo中使用的最主要的一个模板引擎
* QWeb是一个基于XML的模板引擎，用于生成HTML的页面和段落

### 指令
* 模板指令是一些t-作为前缀的XML属性
    * 例子：如果t-if的条件满足，里面的元素和属性都会被直接渲染
* 为了避免元素被渲染，可以使用\<t>占位元素
    * 该元素只会执行指令，但不会产生任何输出

#### 例子
##### 使用\<t>
```xml
<t t-if="condition">
    <p>Test</p>
</t>
```
渲染结果
```html
<p>Test</p>
```

##### 不使用\<t>
```xml
<div t-if="condition">
    <p>Test</p>
</div>
```
渲染结果
```html
<p>Test</p>
```


## 数据输出
### esc
* 该指令是QWeb的主要的输出指令
* 该指令会自动进行HTML转义，当显示用户提交的内容时可以减小XSS的风险
* 示例
    ```xml
    <p><t t-esc="value"/></p>
    ```
    当value值为42时，渲染结果
    ```html
    <p>42</p>
    ```

### raw
* 该指令是另外一个输出指令
* 该指令与esc指令基本相同，除了不会进行HTML转义


## 条件判断
* 条件指令是if，会对给定的属性值进行表达式求值
* 示例
    ```xml
    <div>
        <t t-if="condition">
            <p>ok</p>
        </t>
    </div>
    ```
    条件为true时
    ```html
    <div>
        <p>ok</p>
    </div>
    ```
    条件为false时
    ```html
    <div>
    </div>
    ```
* 条件渲染可以直接应用在指令所有元素上，可以不使用\<t>
    ```xml
    <div>
        <p t-if="condition">ok</p>
    </div>
    ```
* 条件分支：t-elif, t-else
    ```xml
    <div>
        <p t-if="user.birthday == today()">Happy birthday!</p>
        <p t-elif="user.login == 'root'">Welcome master!</p>
        <p t-else="">Welcome!</p>
    </div>
    ```


## 循环
* 遍历指令为foreach，需要传入一个返回可遍历集合的表达式
    * 第二个参数t-as提供了对‘当前项’的使用
* 示例
    ```xml
    <t t-foreach="[1, 2, 3]" t-as="i">
        <p><t t-esc="i"/></p>
    </t>
    ```
    渲染结果
    ```html
    <p>1</p>
    <p>2</p>
    <p>3</p>
    ```
* 该指令也可以直接应用在指令所有元素上，可以不使用\<t>
* 该指令可以遍历数组、映射（当前项为键）
* 其他变量
    * $as_value : 当前的值，对于列表该变量和$as一样，对于映射该变量提供值($as提供键)
    * $as_index : 当前的遍历索引值（从0开始）
    * $as_size : 集合的大小
    * $as_first : 当前项是否是第一项（等于$as_index == 0）
    * $as_last : 当前项是否是是最后一项（等于$as_index + 1 == $as_size）
* 该指令中创建的新变量只在该指令的范围内有效
    * 如果变量在循环外部也存在，在循环结束后值会被拷贝到全部上下文中
    * 示例
        ```xml
        <t t-set="existing_variable" t-value="False"/>
        <!-- existing_variable 现在的值是 False -->

        <p t-foreach="[1, 2, 3]" t-as="i">
            <t t-set="existing_variable" t-value="True"/>
            <t t-set="new_variable" t-value="True"/>
            <!-- existing_variable 和 new_variable 现在的值是 True -->
        </p>

        <!-- existing_variable 一直是 True -->
        <!-- new_variable 未定义 -->
        ```


## 属性

### 概述
* 使用t-att指令，可以联机计算属性，将计算结果设置到输出节点上

### t-att-$name
* 创建一个名字为$name的属性，属性的值会被求值并将结果设置为$name属性的值
* 示例
    ```xml
    <div t-att-a="42"/>
    ```
    渲染结果
    ```html
    <div a="42"></div>
    ```

### t-attf-$name
* 功能类似于t-att-$name，但是参数是一个格式化字符串
* 示例
    ```xml
    <t t-foreach="[1, 2, 3]" t-as="item">
        <li t-attf-class="row {{ (item_index % 2 === 0) ? 'even' : 'odd' }}">
            <t t-esc="item"/>
        </li>
    </t>
    ```
    渲染结果
    ```html
    <li class="row even">1</li>
    <li class="row odd">2</li>
    <li class="row even">3</li>
    ```

### t-att=mapping
* 如果参数是一个映射，每个键值对都会生成一个属性
* 示例
    ```xml
    <div t-att="{'a': 1, 'b': 2}"/>
    ```
    渲染结果
    ```html
    <div a="1" b="2"></div>
    ```

### t-att=pair
* 如果参数是一个数据对（元组、两个元素的数组）
    * 第一个元素会成为属性的名字
    * 第二个元素会成为属性的值



## 设置变量
* 使用set指令可以在模板中创建变量，来保存计算结果、给一块数据一个清晰的名字
* 该指令接收参数来作为变量的名字
* 变量值有两种方法可以设置
    * 使用t-value指令
    * 节点的内容会被渲染，作为变量的值
* 示例
    输出值为3
    ```xml
    <t t-set="foo" t-value="2 + 1"/>
    <t t-esc="foo"/>
    ```

    输出值为&lt;li&gt;ok&lt;/li&gt;
    ```xml
    <t t-set="foo">
        <li>ok</li>
    </t>
    <t t-esc="foo"/>
    ```


## 调用子模板
* QWeb模块可以被用来顶层渲染中
* 通过使用t-call指令，也可以在其他模板中被使用
* 子模板将会在父模板的上下文中被执行
* 示例
    子模板
    ```xml
    <p><t t-value="var"/></p>
    ```
    父模板
    ```xml
    <t t-set="var" t-value="1"/>
    <t t-call="other-template"/>
    ```
    渲染结果
    ```html
    <p>1</p>
    ```
* 在该指令外部会有变量可见性的问题
    ```xml
    <t t-call="other-template">
        <t t-set="var" t-value="1"/>
    </t>
    <!-- "var" 在这个并不存在 -->
    ```
* 通过变量0，可以将指令的内容渲染进子模板中
    子模板
    ```xml
    <div>
        This template was called with content:
        <t t-raw="0"/>
    </div>
    ```
    父模板
    ```xml
    <t t-call="other-template">
        <em>content</em>
    </t>
    ```
    渲染结果
    ```html
    <div>
        This template was called with content:
        <em>content</em>
    </div>
    ```




## Python

### 独占指令
#### 字段格式化
* t-field指令可以用来在一个“智能”记录(browse方法的结果)上进行字段存取
* 该指令也可以基于字段类型来自动格式化字段值
* t-options可以用来自定义字段，最常用的选项是widget，其他都是字段相关、控件相关的


### 调试
* t-debug指令通过使用PDB的set_trace指令来触发调试器
* 示例
    ```xml
    <t t-debug="pdb">
    ```
    等价于：importlib.import_module("pdb").set_trace()


### 辅助函数
#### 基于请求的
* 服务端的QWeb模板都是在控制器中使用的
* 这种情况下，模板是被存储在数据库中的，可以通过调用odoo.http.HttpRequest.render()来渲染
* 示例
    ```python
    response = http.request.render('my-template', {
        'context_value': 42
    })
    ```
    说明：该函数会自动创建一个Response对象，作为控制器的返回值

#### 基于视图的
* ir.ui.view存在一个render方法
* 方法原型：render(cr, uid, id,[values], [engine='ir.qweb'], [context])
* 渲染上下文中的值
    * request: 当前的WebRequest对象
    * debug: 当前请求是否在调试模式
    * quote_plus: url编码函数



## JS

### 独占指令
#### 定义模板
* t-name指令用于定义模板，只能放置在模板文件的顶层位置
* 示例
    ```xml
    <templates>
        <t t-name="模板名字">
            <!-- 模板代码 -->
        </t>
    </templates>
    ```

#### 模板继承
##### 作用
* 就地改变一个已存在的模板
* 从一个父模板中创建一个新的模板

##### 指令
* t-inherit: 值为需要继承的模板的名字
* t-inherit-mode: 定义继承的行为
    * primary: 从父模板中创建一个新的子模板
    * extension: 就地改变父模板
* t-name: 在primary模板下，用于定义新模板的名字

##### 示例
```xml
<t t-name="child.template" t-inherit="base.template" t-inherit-mode="primary">
    <xpath expr="//ul" position="inside">
        <li>new element</li>
    </xpath>
</t>
```

```xml
<t t-inherit="base.template" t-inherit-mode="extension">
    <xpath expr="//tr[1]" position="after">
        <tr><td>new cell</td></tr>
    </xpath>
</t>
```


### 调试
#### t-log
* 该指令传入一个表达式，在渲染时进行求值并使用console.log输出
* 示例
    ```xml
    <t t-set="foo" t-value="42"/>
    <t t-log="foo"/>
    ```

#### t-debug
* 在渲染时触发一个调试器断点
* 当调试器激活后代码将停止执行
* 示例
   ```xml
    <t t-if="a_test">
        <t t-debug="">
    </t>
   ```

#### t-js
* 节点的内容是一段js代码，会在渲染时被执行
* 接收一个上下文参数，可以将渲染上下文带入js代码中
* 示例
    ```xml
    <t t-set="foo" t-value="42"/>
    <t t-js="ctx">
        console.log("Foo is", ctx.foo);
    </t>
    ```


### 辅助函数
#### core.qweb
* 是一个QWeb2.Engine()的实例
* 会加载所有的模块定义的模板文件
* 引入一个标准辅助对象'_', 翻译函数'_t'和JSON