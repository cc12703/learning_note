

# Vue状态管理

## Vuex

* 为Vue开发的状态管理模式
* 集中式的存储了应用中所有组件的状态
* 以相关规则保证状态以一种可预测的方式发生变化


## 总体

* 核心是一个store（仓库），是一个容器包含着应用中的大部分state（状态）
* 状态都是响应式的
* 不能直接改变状态，需要显示的commit（提交）

### 例子
```js
//定义
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  }
})

//读取
console.log(store.state.count)

//修改
store.commit('increment')
```

## State

### 说明
* 使用 **单一状态树** 一个对象包含全部的应用层级状态

### 获取状态

#### 使用计算属性
```js
// 创建一个 Counter 组件
const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return store.state.count
    }
  }
}
```

#### 注入状态
```js
const app = new Vue({
  el: '#app',
  // 把 store 对象提供给 “store” 选项，这可以把 store 的实例注入所有的子组件
  store,
  components: { Counter },
  template: `
    <div class="app">
      <counter></counter>
    </div>
  `
})


const Counter = {
  template: `<div>{{ count }}</div>`,
  computed: {
    count () {
      return this.$store.state.count
    }
  }
}
```

#### mapState
* 一个辅助函数，用于自动从状态中生成计算属性

```js
// 在单独构建的版本中辅助函数为 Vuex.mapState
import { mapState } from 'vuex'

export default {
  
  computed: mapState({
    // 箭头函数可使代码更简练
    count: state => state.count,

    // 传字符串参数 'count' 等同于 `state => state.count`
    countAlias: 'count',

    // 为了能够使用 `this` 获取局部状态，必须使用常规函数
    countPlusLocalState (state) {
      return state.count + this.localCount
    }
  })
}
```

* 如果计算属性名称和状态名字相同时，可以简化
```js
computed: mapState([
  // 映射 this.count 为 store.state.count
  'count'
])
```


## Getter

### 功能
* 类似与vue中的计算属性
* 返回值会被缓存起来

### 例子
```js
const store = new Vuex.Store({
  state: {
    todos: [
      { id: 1, text: '...', done: true },
      { id: 2, text: '...', done: false }
    ]
  },
  
  getters: {
     //定义，一个参数
    doneTodos: state => {
      return state.todos.filter(todo => todo.done)
    }
    
    //定义，两个参数
    doneTodosCount: (state, getters) => {
      return getters.doneTodos.length
    }
    
    getTodoById: (state) => (id) => {
       return state.todos.find(todo => todo.id === id)
    }
  }
  
//通过属性访问  
store.getters.doneTodos


//通过方法访问，结果不会被缓存
store.getters.getTodoById(2)
  
```

### mapGetters
* 一个辅助函数，用于将store中的getter映射到局部计算属性

```js
import { mapGetters } from 'vuex'

export default {
  // ...
  computed: {
  // 使用对象展开运算符将 getter 混入 computed 对象中
    ...mapGetters([
      'doneTodosCount',
      'anotherGetter',
      // ...
    ])
  }
}
```

## Mutation

### 功能
* 提交mutation是更改store中状态的唯一方法
* 每个mutation包括：事件类型 和 回调函数
* mutation必须是同步函数

### 例子
```js
const store = new Vuex.Store({
  state: {
    count: 1
  },
  //定义
  mutations: {
    increment (state) {
      // 变更状态
      state.count++
    }
  }
})

//触发
store.commit('increment')
```

### 提交载荷（Payload）
* 是向store.commit中传入的一个额外参数

```js
// ...
mutations: {
  increment (state, n) {
    state.count += n
  }
}

store.commit('increment', 10)
```

* 载荷最好是一个对象
```js
// ...
mutations: {
  increment (state, payload) {
    state.count += payload.amount
  }
}

store.commit('increment', {
  amount: 10
})

//使用对象风格
store.commit({
  type: 'increment',
  amount: 10
})

```

### 规则
* 提前在store中初始化好所有的需要属性
* 对象上添加心跳属性时，使用 Vue.set(obj, 'newProp', 123)


### 使用常量替换事件类型
```js
// mutation-types.js
export const SOME_MUTATION = 'SOME_MUTATION'


// store.js
import Vuex from 'vuex'
import { SOME_MUTATION } from './mutation-types'

const store = new Vuex.Store({
  state: { ... },
  mutations: {
    // 我们可以使用 ES2015 风格的计算属性命名功能来使用一个常量作为函数名
    [SOME_MUTATION] (state) {
      // mutate state
    }
  }
})
```

### mapMutations
* 辅助函数，用于将methods映射为store.commit()

```js
import { mapMutations } from 'vuex'

export default {
  // ...
  methods: {
    ...mapMutations([
      // 将 `this.increment()` 映射为 `this.$store.commit('increment')`
      'increment', 

      // 将 `this.incrementBy(amount)` 映射为 `this.$store.commit('incrementBy', amount)`
      'incrementBy' 
    ]),
    ...mapMutations({
      // 将 `this.add()` 映射为 `this.$store.commit('increment')`
      add: 'increment' 
    })
  }
}
```


## Action

### 功能
* 提交的是mutation，不是直接变更状态
* 可以包含任意的异步操作
* 使用store.dispatch来触发

### 定义
```js
const store = new Vuex.Store({
  state: {
    count: 0
  },
  mutations: {
    increment (state) {
      state.count++
    }
  },
  actions: {
    increment (context) {
      context.commit('increment')
    }
  }
})
```

### 分发
* 使用mapActions将组件方法映射为store.dispatch

```js
// 以载荷形式分发
store.dispatch('incrementAsync', {
  amount: 10
})

// 以对象形式分发
store.dispatch({
  type: 'incrementAsync',
  amount: 10
})
```

### 组合
* action可以返回promise

```js
actions: {
  actionA ({ commit }) {
    return new Promise((resolve, reject) => {
      setTimeout(() => {
        commit('someMutation')
        resolve()
      }, 1000)
    })
  }
}

//使用
store.dispatch('actionA').then(() => {
  // ...
})

```


## module

* 用于将状态分散到多个对象中