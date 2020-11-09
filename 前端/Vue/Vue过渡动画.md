
# Vue过渡动画

## 总体

* 使用CSS的过渡和动画
* 使用CSS的动画库 （Animate.css）
* 在钩子函数中使用js代码操作DOM
* 使用js动画库（Velocity.js）


## 单元素/组件

### 说明
* 使用transition组件，来添加进入/离开过渡

### 作用范围
* 条件渲染（v-if）
* 条件展示（v-show）
* 动态组件
* 组件根节点

### 例子

**html**
```html
<div id="demo">
  <button v-on:click="show = !show">
    Toggle
  </button>
  <transition name="fade">
    <p v-if="show">hello</p>
  </transition>
</div>
```

**js**
```js
new Vue({
  el: '#demo',
  data: {
    show: true
  }
})
```

**css**
```css
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s;
}
.fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
  opacity: 0;
}
```

### 过渡名

#### 图示
![](https://gitee.com/cc12703/figurebed/raw/master/img/20201109114510.png)

#### 说明
* v-enter 进入的开始状态，元素被插入前生效
* v-enter-to 进入的结束状态，元素被插入后生效
* v-enter-active 进入的生效状态，在整个进入过程生效
* v-leave 离开的开始状态
* v-leave-to 离开的结束状态
* v-leave-ative 离开的生效状态，在整个离开过程生效


### CSS过渡
```css
/* 可以设置不同的进入和离开动画 */
/* 设置持续时间和动画函数 */

.slide-fade-enter-active {
  transition: all .3s ease;
}

.slide-fade-leave-active {
  transition: all .8s cubic-bezier(1.0, 0.5, 0.8, 1.0);
}

.slide-fade-enter, .slide-fade-leave-to {
  transform: translateX(10px);
  opacity: 0;
}
```

### CSS动画
```css
.bounce-enter-active {
  animation: bounce-in .5s;
}
.bounce-leave-active {
  animation: bounce-in .5s reverse;
}

@keyframes bounce-in {
  0% {
    transform: scale(0);
  }
  50% {
    transform: scale(1.5);
  }
  100% {
    transform: scale(1);
  }
}
```


### 钩子函数

#### 说明
* 在transition中声明

#### 例子
```html
<transition
  v-on:before-enter="beforeEnter"
  v-on:enter="enter"
  v-on:after-enter="afterEnter"
  v-on:enter-cancelled="enterCancelled"

  v-on:before-leave="beforeLeave"
  v-on:leave="leave"
  v-on:after-leave="afterLeave"
  v-on:leave-cancelled="leaveCancelled"
>
  <!-- ... -->
</transition>
```