[TOC]

# JavaScript语法

## 总体

* ECMAScript 是 JavaScript 的规范
* ECMAScript包括 ES5（ES2009），ES6（ES2015） 多个版本
* JavaScript 是 ECMAScript 的一种实现


## 变量

### 声明

* var 会有作用域的问题
* let 声明代码块范围的局部变量
* const 声明代码块范围的局部常量


## 数据类型

* 数值
* 字符串
* 布尔值
* 狭义对象
* 数组
* 函数
* undefined  未定义，不含有值
* null 无值


### 数值
* 所有数字都是小数，内部使用64位浮点数保持


### 字符串
* 使用 单引号、双引号 定义
* 推荐使用 === 和 !== 来比较 （比较值和类型）
* 反斜杠（\）用于转义
* 字符串可以被视为字符数组
* length属性返回字符串的长度
* 内部用Unicode存储

### 狭义对象
* 对象是基于prototype，而不是类
* 对象由花括号分隔
* 对象的属性以名称和值对的形式定义，属性由逗号分隔

#### 创建

##### 直接创建
```js
person=new Object();
person.firstname="John";
person.lastname="Doe";
person.age=50;
person.eyecolor="blue";

person={firstname:"John",lastname:"Doe",age:50,eyecolor:"blue"};
```

##### 使用对象构造器
```js
function person(firstname,lastname,age,eyecolor)
{
    this.firstname=firstname;
    this.lastname=lastname;
    this.age=age;
    this.eyecolor=eyecolor;
}

//创建对象
var myMother=new person("Sally","Rally",48,"green");
```

#### 获取所有属性

##### 使用 Object.keys 方法
```js
var o = {
  key1: 1,
  key2: 2
};

Object.keys(o);
// ['key1', 'key2']
```

#### 遍历属性

##### 使用 for…in 循环
* 会跳过不可遍历的属性
* 会遍历对象自身的，和继承的属性

```js
var person={fname:"John",lname:"Doe",age:25}; 

for (x in person)
{
  txt=txt + person[x];
}
```

### 函数

* 函数也是对象
* 使用 var foo = function () { } 形式定义

```js
function myFunction (a,b) {
    return a*b;
}
```


## 数据结构

### array

#### 创建
```js
var cars=new Array();
 cars[0]="Saab";
 cars[1]="Volvo";
 cars[2]="BMW";

var cars=new Array("Saab","Volvo","BMW");
var cars=["Saab","Volvo","BMW"];
```

#### 内建函数
* map() 使用映射函数新建一个数组 array.map( x  => x * 2 )
* map() 可以传入索引号 array.map( (x, index) => x * 2 )
* flatMap() 将callback返回的嵌套数组偏平化 ["my favorite", "is a"].flatMap(x => x.split(" "))
* every() 测试所有元素都符合条件 array.every( x =>  x < 40 )
* some() 测试某些值是否符合条件
* filter() 使用过滤函数新建一个数组 array.filter( word => word.length > 6 )
* find() 查找元素，返回第一个发现的元素 array.find( x > 10 )
* findIndex() 查找元素，返回第一个发现元素的索引，无返回-1
* reduce() 遍历每个元素，将最终结果输出 array.reduce( (x,y))
* forEach() 遍历每个元素 array.forEach( function(value, index, array) {} )
* indexOf() 获取某个元素的位置 array.indexOf('one')
* lastIndexOf() 获取某个元素的位置，从尾部开始
* length = 0 清空数据


### 类型数组

* 在ES6中定义
* 提供了一个基本的二进制数据缓冲区的类数组视图
* 包括 Int8Array，Uint8Array，Int16Array，Int32Array 等等


### map

#### 创建
```js
var first = new Map([
  [1, 'one'],
  [2, 'two'],
  [3, 'three'],
]);
```

#### 内建函数
* size 获取个数
* set(key, val) 增加、更新元素
* delete(key) 删除元素
* clear() 清除元素
* get(key) 获取元素
* has(key) 判断元素是否存在

#### 遍历
```js
for (var key of myMap.keys()) {
  console.log(key);
}

for (var [key, value] of myMap.entries()) {
  console.log(key + ' = ' + value);
}

myMap.forEach(function(value, key) {
  console.log(key + ' = ' + value);
});
```

## 其他

### 正则表达式

#### 语法

* 格式： /pattern/modifiers
* pattern 定义一个模式
* modifiers 定义一个修饰符

##### modifiers值
* i 执行对大小写不敏感的匹配
* g 执行全局匹配
* m 执行多行匹配


#### 字符串方法

##### search() 
* 用于检索字符串中指定的子字符串

```js
var str = "Visit w3cschool"; 
var n = str.search(/w3cschool/i);
```

##### replace() 
* 用于在字符串中用一些字符替换另一些字符

```js
var str = "Visit Microsoft!"; 
var res = str.replace(/microsoft/i, "w3cschool");
```

#### RegExp对象
* 是一个预定义了属性和方法的正则表达式对象

##### test() 
* 用于检测一个字符串是否匹配某个模式

```js
/e/.test("The best things in life are free!")
```

##### exec() 
* 用于检索字符串中的正则表达式的匹配 
* 返回一个数组，其中存放匹配的结果

```js
/e/.exec("The best things in life are free!");
```

### 二进制数据

* ArrayBuffer对象
* TypedArray视图
* DataView视图

#### ArrayBuffer
* 代表存储二进制数据的一段内存，不能直接读写，需要通过视图来读写

##### 操作
* 生成  new ArrayBuffer(len)
* 获取长度 byteLength
* 获取子部分  slice(offset, len)

#### TypeArray
* 使用数组方式来操作ArrayBuffer
* 所有成员，都是同一种类型
* 成员默认值为0



## 最佳实践

### 语法

#### 不要使用==
* 永远只使用 === 和 !== (可以比较值和类型)
* == 和 != 只能比较值相等

#### 不要使用with 
* 运行低效率，而且可能会导致意外


#### 不要使用eval
* 有性能和安全性的问题，并且使得代码更难阅读

#### 函数定义使用固定形式
* 只使用 var foo = function () { } 形式