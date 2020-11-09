

# Vue概述


## 总体

* 一套用于构建用户界面的渐进式框架
* 核心库只关注视图层
* 核心是一个允许采用模板语法来声明式地将数据渲染进DOM的系统


## 实例

* 每个Vue应用都是一个Vue实例，通过Vue函数创建
* 一个Vue应用：根Vue实例 + 组件树
* 所有的Vue组件也都是Vue实例


```js
var vm = new Vue({
  // 选项
})
```


### 数据
* 实例被创建时，data中的property会被加入到响应系统中
* 只有实例创建时，已经存在于data中的property才是响应式的
* 前缀$开头的是Vue系统的property



### 生命周期回调

#### 图示
![](https://gitee.com/cc12703/figurebed/raw/master/img/vue-生命周期.png)


#### 说明
* beforeCreate 实例初始化后，数据观测和事件配置之前
* created  数据观测和事件已配置
* beforeMount 挂载前
* mounted 挂载后，不会保证所有子组件都已经挂载好
* beforeUpdate 数据更新时，虚拟DOM打补丁前
* updated 虚拟DOM重新渲染和打补丁后
* beforeDestroy 实例销毁前
* destroyed 实例被销毁后





## 模块语法

[Vue模板语法.md]

## 组件

[Vue组件.md]


## 动画

[Vue过渡动画.md]


## 过滤器

### 功能
* 自定义过滤器，用于文本格式化

### 使用方式
* 双花括号插值
* v-bind表达式
* 串联使用

```html
<!-- 在双花括号中 -->
{{ message | capitalize }}

<!-- 在 `v-bind` 中 -->
<div v-bind:id="rawId | formatId"></div>

<!-- 两个串联 --
{{ message | filterA | filterB }}
```


### 定义

#### 本地形式
* 在组件中定义本地的过滤器

```js
filters: {
  capitalize: function (value) {
    if (!value) return ''
    value = value.toString()
    return value.charAt(0).toUpperCase() + value.slice(1)
  }
}
```

#### 全局形式
* 在Vue中定义全局的过滤器

```js
Vue.filter('capitalize', function (value) {
  if (!value) return ''
  value = value.toString()
  return value.charAt(0).toUpperCase() + value.slice(1)
})

new Vue({
  // ...
})
```



## 状态管理

[Vue状态管理.md]




