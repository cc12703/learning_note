
[TOC]

# TypeScript语法

## 总体

* 为js提供了可选的类型系统
* 是js的超集，兼容js
* 支持未来js所具有的功能


## 变量

* let 定义变量，使用块作用域，与外界变量相隔离
* const 定义常量



## 数据类型

* string, number, boolean, object, function
* Object, Array
* null, undefined, void
* any


### null、undefined
* null 含义：变量不可用
* undefined 含义：变量没有初始化
* 推荐使用 == null 来检查undefined和null
* 检查变量是否已被定义：typeof someglobal !== 'undefined


#### JSON标准
* 支持编码null，若包含会输出null值
* 不支持编码undefined，若包含会被删除'



### 操作
* typeof 获取变量的数据类型, 返回字符串
* <type> 类型断言，指定某个类型
* 值 as type 类型断言，指定某个类型


### 布尔值
```ts
let isDone: boolean = fales
```

### 数字
* 所有数字都是浮点数
* 类型是 number

```ts
let dec: number = 6
```

### 字符串
* 正常字符串 单引导，双引导 定义
* 模板字符串 反引导 定义

```ts
let name: string = "bob"
let sentence: string = `Hello, my name is ${ name }`
```

### 数组
```ts
let list: number[] = [1, 2, 3]
let list: Array<number> = [1, 2, 3]
```

### 元组
* 各元素的类型可以不一样

```ts
let x: [string, number] = ['hello', 10]

```

### 枚举
* 默认编号从0开始
* 值可以数字、字符串
* 可以使用namespace来增加静态方法

```ts
enum Color {Red, Green, Blue}
enum Color {Red = 1, Green = 2, Blue = 4}

namespace Color {

    function toString() {
        .....
    }

}

let c: Color = Color.Green
```

### 任意值
* 屏蔽编译器的类型检查
* 与Object的区别：仍然可以调用对象上的方法

```ts
let notSure: any = 4
notSure.ifItExists()
notSure.toFixed()
```

### 空值
* 表示没有任何类型
* 用于表示函数没有返回值

```ts
function warnUser(): void {
    alert("This is my warning message")
}
```

## 类

* class 用于定义类
* extends 用于继承，只能实现单继承
* static 用于定义静态成员和函数
* 支持public, private, protected，默认为public
* abstract 用于定义抽象类和函数
* constructor 构造器，用于定义成员变量

```ts
class Point {

    x: number
    y: number
    members = []
    
    constructor(x: number, y: number) {
        this.x = x
        this.y = y
    }
}

class Point3D extends Point {
    z: number
    
    constructor(x: number, y: number, z: number) {
        super(x, y)
        this.z = z
    }
}
```


## 箭头函数

* 胖箭头函数（=>）或者lambda函数
* 不需要使用function关键字
* 可以从函数外部捕获到this的引用


## 模块

* 模块在其自身的作用域里执行，而不是在全局作用域里
* 任何包含顶级import或者export的文件都被当成一个模块
* 两个模块之间的关系是通过在文件级别上使用imports和exports建立的

### 导出

#### 方式1
```ts
export const someVar = 123
export type someType = {
    foo: string
}
```

#### 方式2
```ts
const someVar = 123
type someType = {
    type: string
}
export { someVar, someType }
export { someVar as aDifferentName }
```

### 导入
```ts
import { someVar, someType } from './foo'
import { someVar as aDifferentName } from './foo'

//加载到指定对象上， foo.someVar, foo.someType
import * as foo from './foo'
```


## 声明空间

* 类型声明空间：声明用于当作类型注解使用
* 变量声明空间：声明可以用作变量使用

```ts
class Foo {}   //类型声明，变量声明
interface Bar {}  //类型声明
type Bas = {}     //类型声明
```

### 声明语法
* declare var 声明全局变量
* declare function 声明全局方法
* declare global 扩展全局变量
* export 导出变量
* export default 默认导出




## 其他


### map对象声明
```ts
let map:{[key:number] : Student} = {};
```

### 转成string类型
```ts
let text: string = String(xxx)
```

### 特殊符号
* 变量后使用 ! 表示类型推断排除null, undefined
* 属性，参数中使用 ？表示该项可选


## 异步

### Promise

#### 目的
* 使用同步的代码来替换异步/回调函数的代码


#### 创建
```ts
const promise = new Promise((reslove, reject) => {
    //使用reslove, reject函数控制Promise
});
```

#### 订阅状态
* 通过then函数, catch函数

```ts
promise.then(res => { ... });
promise.caceh(err => { ... });
```

#### 链式调用
* 当Promise的值是resolved时，只有then会被调用
* 可以用单个catch来捕获前面链中抛出的异常
* catch实际上会返回一个新的Promise
* 在then, catch中抛出的任何同步错误都会导致返回Promise失败
* 出现的任何错误都会导致直接进入尾部的catch
* ts可以通过Promise链来理解从Promise传过来的值


### generators

#### 目的
* 创建惰性迭代器
* 外部控制函数执行

#### 说明
* 语法: function* 
* 调用generators函数将返回一个generators对象
* 该对象遵循了迭代器接口（next, return, throw)

#### 运行逻辑
1. 调用对象的next()，函数才会执行
2. 遇到yield函数，函数会暂停执行
3. 再次调用对象的next()，函数会继续执行


### async/await

* 异步函数用async关键字来标记
* await将暂停执行，直到异步函数返回的Promise对象执行完成，并返回内部的值
* 使用了generators来实现


## 类型系统


### 注解

#### 基本类型
* 使用js的基本数据类型
```ts
let num: number
let str: string
let bool: boolean
```

#### 数组类型
```ts
let array: boolean[]
```

#### 元组类型
```ts
let nameNumber: [string, number]
```

#### 接口
```ts
interface Name {
    first: string
    second: string
}

let name: Name
```

#### 内联
```ts
let name: {
    first: string
    second: string
}
```

#### 特殊
* any 类型系统的后门，ts会把类型检查关闭
* null, undefined 能赋予任意类型
* void 无返回值

#### 泛型
```ts
function reverse<T>(items: T[]): T[] { ... }
```

#### 联合类型
```ts
function format(command: string[] | string) {

    if(typeof command === 'string') {
        ...
    }

}
```

#### 类型别名
```ts
type StrOrNum = string | number
type Text = string | { text: string }
type Callback = (data: string) => void
```



### 环境声明

* 允许你安全地使用现有的js库
* 通过declare来向ts表述一个其他环境中已经存在的代码
* 建议将声明放入xxx.d.ts文件中


### 接口

* 接口是开放式的
* 类可以实现接口，使用implements