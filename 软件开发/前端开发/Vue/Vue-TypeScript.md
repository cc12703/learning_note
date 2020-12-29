

# Vue-TypeScript


## 引入TS

### 安装插件
* vue-class-component  使用ts装饰器来增强vue组件
* vue-property-decorator  在vue-class-component基础上继续增强
* ts-loader  使webpack识别.ts, .tsx文件


### 增加tsconfig.json

#### 配置项
* experimentalDecorators 是否启用装饰器
* emitDecoratorMetadata 是否启用设计类型元数据（用于反射）
* removeComments 编译时是否移除注释
* lib 编译时要引入的库文件
* module 采用的模型系统
* moduleResolution 如何处理模块
* target 编译出的目标es版本

### 识别.vue
* 增加vue-shim.d.ts文件
* 导入vue文件时，需要加上.vue后缀


#### 定义内容
```ts
declare module "*.vue" {
  import Vue from "vue";
  export default Vue;
}
```



## 改造vue文件


### vue-property-decorator
* 是在vue-class-component上增强了更多的结合Vue特性的装饰器
* 参考：https://github.com/kaorun343/vue-property-decorator


#### Component

##### 定义组件类
```ts
@Component({
    //导入其他组件
    components: {
        OtherComponent
    }
})
export default class HelloWorld extends Vue {
  //定义一个组件数据，带有响应属性
  message = 'Hello World!'
}
```

##### 定义计算属性
```ts
@Component
export default class HelloWorld extends Vue {
  firstName = 'John'
  lastName = 'Doe'

  // Declared as computed property getter
  get name() {
    return this.firstName + ' ' + this.lastName
  }

  // Declared as computed property setter
  set name(value) {
    const splitted = value.split(' ')
    this.firstName = splitted[0]
    this.lastName = splitted[1] || ''
  }
}
```

#### Prop

##### 定义
```ts
@Component
export default class YourComponent extends Vue {
  @Prop(Number) readonly propA: number | undefined
  @Prop({ default: 'default value' }) readonly propB!: string
  @Prop([String, Boolean]) readonly propC: string | boolean | undefined
}
```

#### Watch

##### 定义一个监听器
```ts
@Component
export default class YourComponent extends Vue {
  @Watch('child')
  onChildChanged(val: string, oldVal: string) {}

  @Watch('person', { immediate: true, deep: true })
  onPersonChanged1(val: Person, oldVal: Person) {}

  @Watch('person')
  onPersonChanged2(val: Person, oldVal: Person) {}
}
```

#### Emit

##### 定义一个事件发送
```ts
@Component
export default class YourComponent extends Vue {
  count = 0

  //事件名 add-to-count
  //事件参数 n
  @Emit()
  addToCount(n: number) {
    this.count += n
  }

  @Emit('reset')
  resetCount() {
    this.count = 0
  }

  //事件名 return-value
  @Emit()
  returnValue() {
    return 10
  }

  @Emit()
  onInputChange(e) {
    return e.target.value
  }

  @Emit()
  promise() {
    return new Promise((resolve) => {
      setTimeout(() => {
        resolve(20)
      }, 0)
    })
  }
}
```


### vuex-module-decorators
* vuex的装饰器
* 参考：https://championswimmer.in/vuex-module-decorators/pages/installation.html


#### Module

##### 定义一个模块
```ts
@Module
export default class MyModule extends VuexModule {
  //定义一个State
  someField: string = 'somedata'
}
```

##### 使用模块
```ts
const store = new Vuex.Store({
  modules: {
    myMod: MyModule
  }
})
```

##### 访问一个State
```ts
this.$store.state.myMod.someField
```

#### Getters

##### 定义
```ts
@Module
export default class Vehicle extends VuexModule {
  wheels = 2
  get axles() {
    return this.wheels / 2
  }
}
```

#### Mutations

##### 定义
```ts
@Module
export default class Vehicle extends VuexModule {
  wheels = 2

  @Mutation
  puncture(n: number) {
    this.wheels = this.wheels - n
  }
}
```