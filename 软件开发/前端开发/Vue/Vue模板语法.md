[TOC]

# Vue模板语法

## 总体

* 使用了基于HTML的模板语法，合法的HTML
* 实现上，Vue会将模板编译成虚拟DOM渲染函数



## 插值

### 文本插值
* \{\{ msg \}\} 标签将会被替代为对应数据对象上的值
* v-once指令：只执行一次插值，数据改变时，插值的内容不会更新

```html
<span>Message: {{ msg }}</span>
<span v-once>这个将不会改变: {{ msg }}</span>
```

### 属性插值
* v-bind指令可以在html属性上进行插值

```html
<div v-bind:id="dynamicId"></div>
<button v-bind:disabled="isButtonDisabled">Button</button>
```

### js表达式
* 每个插值的地方都支持javaScript表达式，都只能包含单个表达式
* 模板表达式都被放在沙盒中，只能访问全局变量的一个子集（白名单）

```html
{{ number + 1 }}

{{ ok ? 'YES' : 'NO' }}

{{ message.split('').reverse().join('') }}

<div v-bind:id="'list-' + id"></div>
```


## 指令

### 组成
* 带有 **v-** 前缀的特殊属性，值是单个javaScript表达式

### 参数
* 在指令名之后以冒号表示
* 只能接收 **一个** 参数

```html
<a v-bind:href="url">...</a>
<a v-on:click="doSomething">...</a>
```

### 修饰符
* 使用半角句号开始
* 指定指令需要以特殊的方式绑定

```html
<form v-on:submit.prevent="onSubmit">...</form>
```

### 缩写

#### v-bind
```html
<!-- 完整语法 -->
<a v-bind:href="url">...</a>

<!-- 缩写 -->
<a :href="url">...</a>
```

#### v-on
```html
<!-- 完整语法 -->
<a v-on:click="doSomething">...</a>

<!-- 缩写 -->
<a @click="doSomething">...</a>
```


## 条件渲染(v-if)

### 功能
* 用于条件性地渲染一块内容
* 这块内容只会在指令的表达式返回true值的时候被渲染

### 要点
* 真正的条件渲染：会确保在切换过程中条件块内的事件监听器和子组件适当地被销毁和重建
* 惰性的：直到条件第一次变为真时，才会开始渲染条件块

### 分组

#### 使用v-if指令
```html
<template v-if="ok">
  <h1>Title</h1>
  <p>Paragraph 1</p>
  <p>Paragraph 2</p>
</template>
```

#### 使用v-else, v-else-if
```html
<div v-if="Math.random() > 0.5">
  Now you see me
</div>
<div v-else>
  Now you don't
</div>


<div v-if="type === 'A'">
  A
</div>
<div v-else-if="type === 'B'">
  B
</div>
<div v-else-if="type === 'C'">
  C
</div>
<div v-else>
  Not A/B/C
</div>
```


### 复用元素
* Vue会尽可能的复用已有的元素 
* 通过使用key属性可以使两个元素完全独立

```html
<template v-if="loginType === 'username'">
  <label>Username</label>
  <input placeholder="Enter your username" key="username-input">
</template>
<template v-else>
  <label>Email</label>
  <input placeholder="Enter your email address" key="email-input">
</template>
```

### 条件显示(v-show)
* 元素会被渲染并保留在DOM中，只是切换元素的CSS属性display
* 不支持复用元素，也不支持v-else

#### 对比
* v-if指令有更高的切换开销
* v-show指令有更高的初始渲染开销

### 遍历元素(v-for)

#### key
* 尽可能在使用v-for时提供 key
* key使Vue可以跟踪每个节点的身份，从而重用和重新排序现有元素

#### 列表渲染
```html
<li v-for="item in items">
   {{ item.message }}
</li>

//index是索引值，从0开始
<li v-for="(item, index) in items">
   {{ parentMessage }} - {{ index }} - {{ item.message }}
</li>
```

#### 遍历元素
```html
<li v-for="value in object">
    {{ value }}
</li>

//key为键名
<li v-for="(value, key) in object">
  {{ key }}: {{ value }}
</li>

//index为索引值
<li v-for="(value, key, index) in object">
  {{ index }}. {{ key }}: {{ value }}
</li>
```
#### 渲染多个元素
```html
<ul>
  <template v-for="item in items">
    <li>{{ item.msg }}</li>
    <li class="divider" role="presentation"></li>
  </template>
</ul>
```

### 监听事件(v-on)

#### 功能
* v-on指令用于监听DOM事件

#### 直接执行js代码
```html
<div id="example-1">
  <button v-on:click="counter += 1">Add 1</button>
  <p>The button above has been clicked {{ counter }} times.</p>
</div>
```

#### 绑定到方法
```html
<div id="example-2">
  <button v-on:click="greet">Greet</button>
</div>

<script>
var example2 = new Vue({
  el: '#example-2',
  data: {
    name: 'Vue.js'
  },

  methods: {
    greet: function (event) {
      alert('Hello ' + this.name + '!')
    }
  }
})
</script>
```

#### 直接调用方法
```html
<div id="example-3">
  <button v-on:click="say('hi')">Say hi</button>
  <button v-on:click="say('what')">Say what</button>
</div>

<script>
new Vue({
  el: '#example-3',
  methods: {
    say: function (message) {
      alert(message)
    }
  }
})
</script>
```

#### 事件修饰符
* .stop 阻止单击事件继续传播
* .prevent 提交事件不再重载页面
* .capture 添加事件监听器时使用事件捕获模式
* .self 只当在 event.target 是当前元素自身时触发处理函数

#### 按键修饰符
* 直接将KeyboardEvent.key暴露的任意有效按键名转换为 kebab-case 来作为修饰符


### 数据绑定(v-model)

#### 功能
* 在表单及元素上创建双向数据绑定 
* 根据控件类型自动选取正确的方法来更新元素

#### 文本
```html
<input v-model="message" placeholder="edit me">
<p>Message is: {{ message }}</p>
```

#### 多行文本
```html
<span>Multiline message is:</span>
<p style="white-space: pre-line;">{{ message }}</p>

<textarea v-model="message" placeholder="add multiple lines"></textarea>
```

#### 修饰符
* .lazy  将同步模式转成使用change事件进行同步
* .number  自动将用户的输入值转为数值类型
* .trim  自动过滤用户输入的首尾空白字符



