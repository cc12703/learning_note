

# VueElementAdmin分析

* [项目文档](https://panjiachen.github.io/vue-element-admin-site/zh/guide/)
* [设计文档](https://juejin.cn/post/6844903476661583880)


## 工具

## 基础

* 跨域问题：使用cors解决
* 前后端接口交互：使用swagger生成文档




## mock数据

### 方法
* 使用mockjs生成mock数据
* 本地启动一个mock-server来模拟数据

### 具体实现
```
/mock/mock-server.js  本地服务
/xxx.js  mock数据
```


## 权限控制

### 登录

#### 流程
1. 用户填写用户名和密码，提交服务端验证
1. 验证通过后服务端返回一个token，前端保存该token
1. 前端使用token获取用户的详细信息

#### 要点
* 登录和获取用户信息是分离的，使用两个接口：login，get_user_info
* 登录成功后，在router.beforeEach()中通过token获取用户信息


### 权限

#### 原理
* 前端有一份路由表，配置每个路由可访问的权限
* 用户登录后，通过token获取到role，根据role动态算出对应的路由
* 通过router.addRoutes动态挂载路由

#### 流程
1. 创建vue实例时，使用vue-router挂载一些公用页面
1. 用户登录后获取role，将role和路由表中的权限进行比较生成最终的用户路由表
1. 调用router.addRouters()添加路由
1. 使用vuex管理路由表，根据vuex中的路由渲染侧边栏

#### 具体实现
* router.js：定义通用路由表(constantRouterMap)和动态路由表(asyncRouterMap)
* main.js：在router.beforeEach()中完成逻辑
* permission.js：在GenerateRoutes()中根据role生成用户路由表
* 侧边栏：使用element-ui的NavMenu来实现


## 图标管理

### 使用iconfont - symbol方式

#### 流程
1. 拷贝生成的symbol代码，引入./iconfont.js
1. 加入通用css代码
    ```css
    <style type="text/css">
        .icon {
        width: 1em; height: 1em;
        vertical-align: -0.15em;
        fill: currentColor;
        overflow: hidden;
        }
    </style>
    ```
1. 选择对应图标，并应用于页面
    ```xml
    <svg class="icon" aria-hidden="true">
        <use xlink:href="#icon-xxx"></use>
    </svg>
    ```


### 优化

#### 创建icon-component

##### 封装
```vue
//components/Icon-svg
<template>
  <svg class="svg-icon" aria-hidden="true">
    <use :xlink:href="iconName"></use>
  </svg>
</template>

<script>
export default {
  name: 'icon-svg',
  props: {
    iconClass: {
      type: String,
      required: true
    }
  },
  computed: {
    iconName() {
      return `#icon-${this.iconClass}`
    }
  }
}
</script>

<style>
.svg-icon {
  width: 1em;
  height: 1em;
  vertical-align: -0.15em;
  fill: currentColor;
  overflow: hidden;
}
</style>
```

##### 使用
```js
//引入svg组件
import IconSvg from '@/components/IconSvg'

//全局注册icon-svg
Vue.component('icon-svg', IconSvg)

//在代码中使用
<icon-svg icon-class="password" />
```


### 由svg-sprite加载svg

#### 配置loader
```json
{
    test: /\.svg$/,
    loader: 'svg-sprite-loader',
    include: [resolve('src/icons')],
    options: {
        syboId: 'icon-[name]'
    }
}
```

#### 使用
```js
import '@/src/icons/qq.svg'; //引入图标

<svg><use xlink:href="#qq" /></svg>  //使用图标
```

### 自动导入

#### 流程
1. 创建放置图标的文件夹：@/src/icons
1. 使用require.context来导入
    ```js
    const requireAll = requireContext => requireContext.keys().map(requireContext)
    const req = require.context('./svg', false, /\.svg$/)
    requireAll(req)
    ```