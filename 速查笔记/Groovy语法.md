


# Groovy语法


## 总体

* 动态类型
* 默认的修饰符为 public 
* 自动导入常用包：  java.lang, java.util, java.io, java.net, groovy.lang, groovy.util
* 静态方法内可以使用this来引用： this指向Class对象
* 不强制处理异常
* 静态导入支持别名：import static Match.random as rand
* 安全导航操作符  **?.** ： 只有对象引用不为空时才会分派调用
* 函数参数可以使用默认值
* 初始化对象时，可以使用逗号分隔的KV



## 数据类型

### 字符串
- 单引号：常量，不会求值其中的表达式，类型为String
- 双引号：表达式，支持惰性求值， ${expression}，类型为GString
- 三个单引号：创建多行字符串
- ==等价于Java的equals()


### Range类
* 定义(包括尾部)： 1..10
* 定义()不包括尾部):   1..<10
* 可以使用集合类的通用方法
* from :  获取开始元素
* to ： 获取结束元素

### 集合类
- 创建实例：[1, 2, 3]
- 读取元素：list[1]  list[2..5]
- each : 迭代元素，传入一个闭包
- collect：操作元素，并返回一个结果集合
- find：查找元素，只找一个
- findAll：查找元素，找所有元素
- sum：将元素相加，并返回结果
- join：使用指定的字符连接元素
- inject：对元素调用闭包
- collectEntries：操作元素，并返回一个map

```groovy
bikes = [ new Bike(name:'xxx', brand:'xxx'), 
          new Bike(name:'xxx', brand:'xxxx') ]
def bikeMap = bikes.collectEntries { [it.name, it.brand] }

println lst.inject(0) { carryOver, elm -> carryOver + elm.size() }
```


### Map类
- 创建实例：[k1:v1, k2:v2, k3:v3]
- 访问键的值：map[k1] 或 map.k1 或  map.'k1'
- each：迭代元素，传入一个闭包
- collect：操作元素，并返回一个结果列表
- any：查找至少一个满足条件的元素
- every：检查是否所有元素都满足条件
- collectEntries：操作元素，并返回一个结果map

```groovy
	{ entry -> $entry.key   $entry.value }
	{ key, value -> .... }

	map.collectEntries { key, value -> [key, value]}
```


## 代码生成

* **编译器会识别选定的注解并生成相应代码**

### @Canonical
* 生成toString方法，可以简单的显示逗号分隔的字段值


### @Delegate
* 用于生成委托代码，即该类中没有被委托类的方法，就把这些方法引入进来

```groovy
class Worker
class Expert
class Manager {
	@Delegate Expert = new Expert()
	@Delegate Worker = new Worker()
}
```

### @Immutable
* 将类中所有的字段标记为final


### @Lazy
* 实现延迟初始化

```groovy
class Heavy {
...
}

class AsNeeded {
	@Lazy Heavy heavy1 = new Heavy()
}
```

### @Singleton
* 实现单例模式


### @Newify
* 创建实例时不使用new关键字，用于创建DSL
* 创建Rudy分格的构造器

```groovy
@Newify([Person, CreditCard])
def fluentCreate() {
	println Person(first:'Json', last:'Doe')
}
```




## 实现接口

* **使用as操作符 和 代码块 来实现**

### 拦截对接口中的任何方法的调用，将其路由到代码块
```groovy
	button.addActionListener (
		{ ... code ... } as ActionListener
	)
```


### 只拦截指定的方法调用
```groovy
	handleFocus = [
		focusGained ： { ... code ... }	
		focusLoast : { ... code ... }
	]
	button.addActionListener ( handleFocus as FocusListener)
```



## 其他



### 操作符重载
* **将每个操作符都映射到一个标准的方法**

#### 例子
- 字符串的 ++  映射为 String类的next方法
- 字符串的 for-each  映射为 String类的next方法
- 集合的 <<  映射为 Collection类的leftShift方法



### Object扩展
- dump：打印对象的详细信息
- inspect：说明创建一个对象需要提供什么
- with：创建一个上下文，闭包中的任何方法调用都会被自动解析到该上下文对象
-  [  ] : 间接访问属性
-  invokeMethod ：间接调用方法

```groovy
car[name] --> car.getAt(name)   //获取属性值
cat[name] = val --> car.putAt(name, val)    //设置属性值
```


### java.io扩展
- eachLine :  对一行调用一次闭包
- filterLine : 过滤文件
- withxxx :  使用完成后自动刷新并关闭一个输入流

```groovy
new File('output.txt').withWriter { file ->
	file << 'some data'
}
```


### 正则表达式
- ~“hello”   创建一个Pattern实例
- /\d*\w*/   创建一个RegEx实例
- =~ 执行部分匹配，返回一个Matcher对象
- ==~ 执行精确匹配，返回一个boolean值






