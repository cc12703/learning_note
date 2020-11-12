

# Kotlin语法




## 特征

* 强类型的语言
* 静态类型语言
* 支持类型推导功能
* 支持函数式编程风格
* 开源并免费的

## 基础

### 变量

* 一切都是对象
* var (可变引用) 变量的值可以被改变
* val (不可变引用) 变量的值不可以在初始化之后再次赋值


### 表达式函数体
* 函数体由单个表达式构成，去掉了花括号和return

```kotlin
fun max(a: Int, b: Int): Int = if (a>b) a else b
```

### 字符串模板
* 求值 ${xxxx}

```kotlin
println("Hello, $name")

println("Hello, ${if (args.size > 0) args[0] else "someone"}!")
```

### 三元表达式
* 可以使用if-else语句来实现：if(true) 1 else 0

### 属性
* 声明和变量一样
* 属性：字段 + 访问器
* var  可写属性： 一个字段 + 一个getter + 一个 setter
* val  只读属性：一个字段 + 一个getter
* 若名字以is开头，getter不增加前缀，setter会将is替换成set

### 目录和包
* 可以把多个类放在同一个文件中，文件名字和随意选择

### 枚举
```kotlin
enum class Color(val r: Int, val g: Int, val b: Int) {
		RED(255,0,0),
		YELLOW(255,255,0),
		BLUE(0,0,255),
		GREEN(0,255,0);

		fun rbg() = (r * 256 + g) * 256 + b
}
```

### when
* 可以使用任何对象
* 不需要break语句
* 如果没有其他分支满足条件，else分支会执行

```kotlin
fun get(color: Color) {
	when(color) {
		Color.RED -> "RED"
		Color.YELLOW, Color.GREEN -> "warm"
		else -> throw Exception("info")
	}
}
```

### 智能转换
* 使用is来检查后，不需要在进行转换了。编译器已经执行了类型转换

```kotlin
fun eval(e: Expr): Int =
	when(e) {
		is Num -> {
			...
		}
		is Sum -> {
			...
		}
	}
```

### 区间
* 一个开始值，一个结束值，中间使用 .. 运算符
* 是闭合的，第二个值是包含在区间内的
* 使用in检查是否在区间中

```kotlin
	val oneToTen = 1..10
	c in oneToTen
	c !in oneToTen
```

### 迭代
```kotlin
//遍历map
for ( (key, val) in map) {
}

//遍历list
for( (index, elm) in list.withIndex()) {
}
```


### 标签
* 任何表达式都可以用标签来标记
* 格式：标识符 + @
* 用来控制return, break, continue的跳转

### 异常
* throw 和 try 都可以作为一个表达式
* 类型是Nothing，表示无返回的函数
* 不区分受检异常和未检异常

### this
* 若无限定符，指的是最内层的包含它的作用域
* 使用 this@label 来引起其他作用域中的this

### 操作符重载
| 表示式      |     函数 |  
| :-------- | :--------:|
| +a   |  a.unaryPlus() |  
| !a  |  a.not()  |
|  a++  |  a.inc() 返回值是a  |
| ++a   |  a.inc() 返回值是 a+1 |
| a in b |  b.contains(a)  |
| a !in b | !b.contains(a)  |
| a[i]  |  a.get(i)  |
| a[i] = b |  a.set(i, b)  |
| a()   |   a.invoke() |
| a(i)  |   a.invoke(i)  |
| a > b  |  a.compareTo(b)  > 0 | 


### 相等
* 引用相等  （===）
* 结构相等，使用equals判断  (==)

### Elvis操作符
* 用于null安全性检查

```kotlin
y = x?:0
val y = if(x !== null) x else 0  //等价于
```


## 空指针安全

* **使用显示的null，强制在必要时进行null检查**
* 一个非空引用不能直接赋值为null
* 类型后加"?"符号，表示变量为可空
* 可空变量可以使用安全调用( ?. ) 和非空断言调用( !!. )

```kotlin
na?.length  //返回null
na!!.length // 返回KotlinNullPointerException
```

## 数据类型

### 根类型Any
* Any运行时会自动映射成java.lang.Object
* 只有 equals(), hashCode(), toString() 三个方法

### 数字类型
* 都继承于Number 和 Comparable 类
* 没有隐式拓宽转换，需要显示转换

### 字符类型
* 使用Char类型表示

### 布尔类型
* 使用Boolean类型表示

### 字符串类型
* 使用String类型表示
* 模板表达式：${xxx}

### 数组类型
* 使用Array类来表示
* 使用arrayOf()来创建一个数组
* 使用arrayOfNulls() 来创建一个空数组
* 同一个数组中，运行不同类型的元素
* 原生数组类型：BooleanArray, ByteArray, CharArray, IntArray

### 可空类型
* 根类型是 Any? 

### Unit类型
* 表示一个函数没有返回值。和java的void功能一样
* 其表达式计算结果的返回类型是Unit

### Nothing类型
* 表示没有实例的类型
* 其表达式计算结果是永远不会返回的
* 类型层次：Nothing  --> Unit  -->  Any

### 类型检测
* 使用 is 来实现
* 编译器会自动插入转换

### 类型转换
* 使用 as 运算符


## 集合

* 分成 可变集合(Mutable) 和 不可变集合(Immutable)
* 类型有：list, set, map

### list
* 创建不可变List :  listOf()
* 创建可变List :    mutableListOf()
* 遍历： forEach{}

#### 操作函数
|函数      |     功能 |  
| :-------- | :--------:|
|  add   | 添加  |
|  remove |  删除   |
| retainAll |  取两个集合的交集  |
| single  |  若集合中只有一个元素，则返回该元素 |
| single(predicate)  | 返回符合条件的单个元素  |
|  any   |   判断集合至少有一个元素 |
| any(predicate)  |  判断集合中是否有满足条件的元素 |
| all(predicate)  |  判断集合中的元素是否都满足条件 |
| none    |   判断集合无元素 |
| none(predicate)  |  判断集合中所有元素都不满足条件 |
| count      |  计算集合中元素的个数  |
| count(predicate)   | 计算集合中满足条件的元素的个数 |
|  reduce    |  从第一个项到最后一项进行累计运算  |
|  forEach(action)  | 遍历元素  |
|  forEachIndexed   | 带下标的遍历元素 |
|  max   |  查询最大元素  |
|  min   |  查询最小元素  |
|  map(transform) |  将集合中的元素进行映射，并将结果保存到一个集合中 | 
|  flatMap(transform)  | 将集合中的元素映射成子集合，并将结果合并成一个集合 |
|  reversed    |   倒序排列  |




### set
* 创建不可变Set:  setOf()
* 创建可变Set:   mutableSetOf()
* 判断对象是否重复:   hashCode()  equals()
* 创建java中的集合类：hashSetOf()  linkedSetOf()   sortedSetOf()


### map
* 创建一个只读空map： mapOf<T, R>()
* 用二元组创建一个只读map： mapOf(pair)
* 创建一个可变的空map： mutableMapOf<T, R>()
* 创建java中的映射类：hashMapOf()   linkedMapOf()  sortedMapOf()


## 函数

* 参数可以定义默认值
```kotlin
fun joinToStr(
		collection: Collection<T>,
		separator: String = ",",
		prefix: String = "",
		postfix: String = ""
		): String
```

* 扩展其他类的函数
* 支持中缀调用


## 泛型

* 泛型主要用于限制集合类持有的对象类型，使得类型更加安全
* 泛型实现都使用了运行时类型擦除方式，对于语法的约束是在编译阶段