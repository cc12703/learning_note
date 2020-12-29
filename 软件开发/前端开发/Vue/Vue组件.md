
[TOC]

# Vue组件

## 基础

* 组件是可复用的 Vue 实例，且带有一个名字
* 组件的data必须是一个函数，保证每个实例有独立的拷贝
* 一个应用会以一棵嵌套的组件树来组织
* 组件注册：全局注册、局部注册
* prop用于向子组件传递数据
* 每个组件必须只有一个根元素
* v-on用于监听子组件事件，$emit用于子组件发送事件
* 使用插糟分发内容


```js
Vue.component('button-counter', {
  data: function () {
    return {
      count: 0
    }
  },
  template: '<button v-on:click="count++">You clicked me {{ count }} times.</button>'
})
```


## 注册

### 组件名

* kebab-case (短横线分隔命名)
* PascalCase (首字母大写命名)

### 全局注册

* 注册后可以用在任何新创建的Vue根实例的模板中

```js
//定义
Vue.component('my-component-name', {
  // ... 选项 ...
})


//使用
new Vue({ el: '#app' })

<div id="app">
  <component-a></component-a>
  <component-b></component-b>
  <component-c></component-c>
</div>
```

### 局部注册

```js
//定义，通过一个普通的js对象
var ComponentA = { /* ... */ }
var ComponentB = { /* ... */ }
var ComponentC = { /* ... */ }

//使用
new Vue({
  el: '#app',
  components: {
    'component-a': ComponentA,
    'component-b': ComponentB
  }
})
```

### 局部注册（模块系统中）

* 创建components目录，每个组件放置在各自的文件中

```js
import ComponentA from './ComponentA'
import ComponentC from './ComponentC'

export default {
  components: {
    ComponentA,
    ComponentC
  },
}
```

## prop

* 作用：用于向组件传递数据，会变成组件实例的属性
* 可以定义类型，默认值，检验函数


### 语法

```js
//定义
Vue.component('blog-post', {
  props: ['title'],
  template: '<h3>{{ title }}</h3>'
})

//使用
<blog-post title="My journey with Vue"></blog-post>
<blog-post title="Blogging with Vue"></blog-post>
<blog-post title="Why Vue is so fun"></blog-post>

```

```js
Vue.component('my-component', {
  props: {
    // 基础的类型检查 (`null` 和 `undefined` 会通过任何类型验证)
    propA: Number,
    // 多个可能的类型
    propB: [String, Number],
    // 必填的字符串
    propC: {
      type: String,
      required: true
    },
    // 带有默认值的数字
    propD: {
      type: Number,
      default: 100
    },
    // 带有默认值的对象
    propE: {
      type: Object,
      // 对象或数组默认值必须从一个工厂函数获取
      default: function () {
        return { message: 'hello' }
      }
    },
    // 自定义验证函数
    propF: {
      validator: function (value) {
        // 这个值必须匹配下列字符串中的一个
        return ['success', 'warning', 'danger'].indexOf(value) !== -1
      }
    }
  }
})
```

### 选项

* type 类型
* default 默认值
* required 是否必须
* validator 校验函数

### 名字

* HTML中的特性名是大小写不敏感的，浏览器会将所有大写字符解释为小写字符 
* 在HTML中使用模板时，camelCase名字需要转换成kebab-case

```js
Vue.component('blog-post', {
  // 在 JavaScript 中是 camelCase 的
  props: ['postTitle'],
  template: '<h3>{{ postTitle }}</h3>'
})

<blog-post post-title="hello!"></blog-post>
```

### 单向数据流

* 所有的prop都使得其父子prop之间形成了一个单向下行绑定
* 父级prop的更新会向下流动到子组件中，但是反过来则不行




## 事件

* 用于组件向外部通知状态变化
* 使用emit来触发，参数：事件名、事件数据
* 使用v-on来监听事件

```js
//触发
<button v-on:click="$emit('enlarge-text'，0.1)">
  Enlarge text
</button>


//接收
<blog-post
  ...
  v-on:enlarge-text="postFontSize += $event"
></blog-post>
```

### 事件名

* 不同于组件和prop，事件名不存在任何自动化的大小写转换。需要完全匹配 
* 建议使用kebab-case (短横线分隔命名)


### 定义v-model

* 需要定义一个名为value的prop 
* 需要触发一个input事件

```js
Vue.component('custom-input', {
  props: ['value'],
  template: `
    <input
      v-bind:value="value"
      v-on:input="$emit('input', $event.target.value)"
    >
  `
})
```

#### 等价转换
```html
<custom-input v-model="searchText"></custom-input>

//等价于
<custom-input
  v-bind:value="searchText"
  v-on:input="searchText = $event">
</custom-input>
```


## 插槽

* 用于组件分发内容
* 插槽内可以包含任何模板代码，包括HTML，其他组件

```js
//定义
Vue.component('alert-box', {
  template: `
    <div class="demo-alert-box">
      <strong>Error!</strong>
      <slot></slot>
    </div>
  `
})

//使用
<alert-box>
  Something bad happened.
</alert-box>
```

### 编译作用域

* 父级模板里的所有内容都是在父级作用域中编译的 
* 子模板里的所有内容都是在子作用域中编译的


### 默认值

* 可以给插槽设置默认内容

```html
<button type="submit">
  <slot>Submit</slot>
</button>
```


## 计算属性

### 功能

* 模板内的表达式只能用于简单运算，对于复杂逻辑都应该使用计算属性


### 语法

```html
<div id="example">
  <p>Original message: "{{ message }}"</p>
  <p>Computed reversed message: "{{ reversedMessage }}"</p>
</div>

<script>
var vm = new Vue({
  el: '#example',
  data: {
    message: 'Hello'
  },
  computed: {
    // 计算属性的 getter
    reversedMessage: function () {
      // `this` 指向 vm 实例
      return this.message.split('').reverse().join('')
    }
  }
})
</script>
```


### 数据缓存

* 是基于它们的响应式依赖进行缓存的
* 只在相关响应式依赖发生改变时它们才会重新求值


### setter函数

* 计算属性默认只有getter函数

```js
computed: {
  fullName: {
  
    // getter
    get: function () {
      return this.firstName + ' ' + this.lastName
    },
    
    // setter
    set: function (newValue) {
      var names = newValue.split(' ')
      this.firstName = names[0]
      this.lastName = names[names.length - 1]
    }
    
  }
}
```

## 侦听器

### 功能
* 用于在数据变化时执行异步或开销较大操作时使用

### 语法
```html
<div id="watch-example">
  <p>
    Ask a yes/no question:
    <input v-model="question">
  </p>
  <p>{{ answer }}</p>
</div>

<script>
var watchExampleVM = new Vue({
  el: '#watch-example',
  data: {
    question: '',
    answer: 'I cannot give you an answer until you ask a question!'
  },
  watch: {
    // 如果 `question` 发生改变，这个函数就会运行
    question: function (newQuestion, oldQuestion) {
      this.answer = 'Waiting for you to stop typing...'
    }
  }
})
</script>
```

### 选项
* deep 是否监听对象内部的变化
* immediate 是否立即触发回调



## 单文件组件

* 扩展名 .vue

```html
<template>
  <p>{{ greeting }} World!</p>
</template>

<script>
module.exports = {
  data: function () {
    return {
      greeting: 'Hello'
    }
  }
}
</script>

<style scoped>
p {
  font-size: 2em;
  text-align: center;
}
</style>
```


## 动态组件

### 功能
* 通过<component>元素的is属性来实现
* 使用组件的名字 或者 组件的选项对象

```html
<component v-bind:is="currentTabComponent"></component>
```

### 缓存组件

* 使用<keep-live>来缓存组件

```html
<!-- 失活的组件将会被缓存！-->
<keep-alive>
  <component v-bind:is="currentTabComponent"></component>
</keep-alive>
```

## 异步组件

### 功能
* 使用工厂函数的方式定义组件
* 该函数会异步解析组件定义
* Vue会在组件需要被渲染时才会触发该函数


### 语法

#### 基础
```js
Vue.component('async-example', function (resolve, reject) {
  setTimeout(function () {
    // 向 `resolve` 回调传递组件定义
    resolve({
      template: '<div>I am async!</div>'
    })
  }, 1000)
})
```

#### webpack
```js
Vue.component('async-webpack-example', function (resolve) {
  // 这个特殊的 `require` 语法将会告诉 webpack
  // 自动将你的构建代码切割成多个包，这些包
  // 会通过 Ajax 请求加载
  require(['./my-async-component'], resolve)
})
```

#### 返回promise
```js
Vue.component(
  'async-webpack-example',
  // 这个动态导入会返回一个 `Promise` 对象。
  () => import('./my-async-component')
)
```