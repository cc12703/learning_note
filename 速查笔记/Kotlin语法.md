

# Kotlin语法

[TOC]


## 基础语法

### 类型声明
* 类型名在变量名的后面
* 编译器可以自动推导出需要的类型
```kotlin
val a: String = "I am Java"
```


### 变量
* 一切都是对象
* var (可变引用) 变量的值可以被改变
* val (不可变引用) 变量的值不可以在初始化之后再次赋值


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

### 标签
* 任何表达式都可以用标签来标记
* 格式：标识符 + @
* 用来控制return, break, continue的跳转


### 函数
#### 概述
* 支持高阶函数：以其他函数作为**参数**或**返回值**的函数
* 支持局部函数：在一个函数内部定义函数
* 支持匿名函数：不定义函数名
* 支持扩展函数：在不修改已有类的前提下，给其增加新方法
* 支持内嵌函数：通过增加inline前缀

```kotlin
fun foo(x: Int) {
	fun double(y: Int): Int {
		return y * 2
	}
	println(double(x))
}

fun(country: Country): Boolean {
	return country.continient == "EU" && countery.population > 10000
}

fun View.invisible() {
	this.visibility = View.INVISIBLE
}

```

#### 类型定义
* 通过->来分隔参数类型和返回值类型
* 用括号来包裹参数类型
* 返回值必须显式声明
```kotlin
() -> Unit 
(Int, String) -> Unit 
(errCode: Int, errMgr: String) -> Unit
(Int) -> ((Int) -> Unit) //返回一个函数
((Int) -> Int) -> Unit   //传入一个函数类型的参数
```

#### 方法引用
* 使用双引号(::)对某个类的方法进行引用
```kotlin
class Book(val name: String)

fun main() {
	val bookName = listOf(
		Book("Test one"),
		Book("Test two")
	).map(Book::name)
}
```

#### Lambda表达式
* 是一种语法糖
* 简化表达后的匿名函数
```kotlin
val sum = { x: Int, y: Int -> x + y }

fun foo(int: Int) = {
	print(int)
}
listOf(1,2,3).forEach { foot(it) }
```

#### 闭包
* 由一对花括号包裹的代码块 并且 访问了外部环境变量
* 可以被当作参数传递或者直接使用


### 字符串
#### 模板
* 求值 ${xxxx}
```kotlin
println("Hello, $name")

println("Hello, ${if (args.size > 0) args[0] else "someone"}!")
```

#### 原生
* 使用3个引号(""")定义
* 最终打印格式与代码中的格式一样，不会进行转义处理
```kotlin
val html = """
		<html>
			<body>
				<p>Hello Word</p>
			</body>
		</html>
		"""
```

#### 判断相等
* === 判断引用相等
* ==  判断内容相等，使用equals进行判断 


### 表达式 
#### 概述
* 表达式是可以返回值的语句
* 目的是为了创造新值
```kotlin
1 + 0
listOf(1,2,3)
"kotlin".length
{ x:Int -> x + 1 }          // Lambda表达式
fun(x: Int) { println(x) } //匿名函数表达式
if (x > 1) x else 1       //if-else表达式
```

#### 三元表达式
* 可以使用if-else语句来实现：if(true) 1 else 0

#### 函数体表达式
* 函数体由单个表达式构成，去掉了花括号和return
* fun后面带有等号(=)
```kotlin
fun max(a: Int, b: Int): Int = if (a>b) a else b
```

#### when表达式
* 每个分支由->连接，不需要break语句
* 由上到下匹配，一直到匹配完，否则执行else分支
* 每个分支都具有返回值

```kotlin
//Color是枚举
fun get(color: Color) {
	when(color) {
		Color.RED -> "RED"
		Color.YELLOW, Color.GREEN -> "warm"
		else -> throw Exception("info")
	}
}
```

#### 范围表达式
* 一个开始值，一个结束值，中间使用 .. 运算符
* 是闭合的，第二个值是包含在区间内的
* in检查是否在区间中
* downTo进行倒序循环
```kotlin
val oneToTen = 1..10
c in oneToTen
c !in oneToTen
for (i in 1..10 step 2) print(i)
for (i in 10 downTo 1 step 2) print(i)
```

#### 中缀表达式
* 通过infix进行定义的函数
	* 函数只能有一个参数，不能有默认值
* 使用格式：A 中缀方法 B

```kotlin
class Person {
	infix fun called(name: String) {
		...
	}
}

val p = Person()
p called "Shaw"

```


### Lambda简化表达
#### 调用SAM
* Java的SAM（单一抽象方法）可以用kotlin函数进行替换
```java
view.setOnClickListener(new OnClickListener() {
	public void onCLick(View view) { }
})
```
简化为
```kotlin
//fun setOnClickListener(listener: (view) -> Unit)
view.setOnClickListener { 
	...
}
```

#### 带接收者
```kotlin
val sum: Int.(Int) -> Int = { other -> plus(other) }

2.sum(1)
```

#### with
```kotlin
fun bindData(bean: ContentBean) {
	with(bean) {
		titleTV.text = this.title //this表示bean，可以省略
	}
}
```

#### apply
```kotlin
fun bindData(bean: ContentBean) {
	bean.apply { 
		titleTV.text = this.title //this表示bean，可以省略
	}
}
```


### 异常
* throw 和 try 都可以作为一个表达式
* 类型是Nothing，表示无返回的函数
* 不区分受检异常和未检异常


### 迭代
```kotlin
//遍历map
for ( (key, val) in map) {
}

//遍历list
for( (index, elm) in list.withIndex()) {
}
```



## 集合

### 概述
* 分成 可变集合(Mutable) 和 不可变集合(Immutable)
* 类型有：list, set, map

### 创建
|函数      |     功能 |  
| :--- | :---|
| listOf() | 创建不可变list |
| mutableListOf() | 创建可变list | 
| setOf() | 创建不可变Set |
| mutableSetOf() | 创建可变Set |  
| mapOf<T, R>() | 创建一个只读空map  |
| mapOf(pair) |用二元组创建一个只读map |
| mutableMapOf<T, R>() | 创建一个可变的空map |

### 高阶API
#### map遍历操作
* 作用：对集合中的元素进行操作，将结果保存到一个新集合中
```kotlin
val newList = list.map { it * 2 }
val newList = list.map { el -> el * 2 }
```

#### filter筛选操作
* 作用：对集合中的元素进行判断，将满足条件的保存到一个新集合中
* filterNot 过滤掉满足条件的元素
* filterNotNull 过滤掉值为null的元素
```kotlin
val mStudents = students.filter { it.sex == "m" }
```

#### sum,sumBy求和操作
* sum 只能对数值列表进行求和
```kotlin
val scoreTotal = students.sumBy { it.score }
val total = listOf(1,2,3,4).sumBy { it }
val total = listOf(1,2,3,4).sum()
```

#### fold
* 作用：对集合中的元素进行遍历，对每个元素调用operation函数
* operation函数传入 上次调用的结果（第一次调用传入initial）和 当前元素
* reduct功能和fold类似，不需要传入initial
```kotlin
val scoreTotal = students.fold(0) { 
	accumulator, student -> accumulator + student.score
}
listOf(1,2,3,4).fold(1) { mul, item -> mul * item }
```

#### groupBy分组操作
```kotlin
//结果类型为 Map<String, List<Student>>
students.groupBy { it.sex }
```

#### flatMap,flatten扁平化操作
* flattern 直接将嵌套的集合进行扁平化，生成一个新集合
* flatMap 在扁平化时，会对元素进行处理
```koltin
list.flatMap { it.map { it.name } }
//等同于
list.flatten().map { it.name }
```

#### 其他操作
|函数      |     功能 |  
| :-------- | :--------:|
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
|  forEach(action)  | 遍历元素  |
|  forEachIndexed   | 带下标的遍历元素 |
|  max   |  查询最大元素  |
|  min   |  查询最小元素  |
|  reversed    |   倒序排列  |


### 惰性求值
* 使用序列进行惰性求值
	* 优化性能
	* 构造出无限的数据类型
* 中间操作：采样惰性求值，返回的都是一个序列，知道如果变换元素
* 末端操作：返回一个结果，会触发中间操作的求值
```kotlin
list.asSequence()
	.filter {it > 2}.map {it * 2}
	.toList()
```

## 面向对象

### 特点
* 类属性需要显示的指定默认值
* 类成员默认是全局可见的
* 接口中的方法可以带默认实现
* 构建对象时，不需要new关键字
* 可以把多个类放在同一个文件中，文件名字和随意选择
```kotlin
class Bird {
	val weight: Double = 500.0
	val color: String = "blue"
	val age: Int = 1
}

val bird = Bird()

interface Flyer {
	val speed: Int

	fun kind()
	fun fly() {
		println("I can fly")
	}
}
```

### 可见性修饰符
* public 默认值，全局可见
* protected 类及子类可见
* private 只有本类可见
* internal 模块内可见（对一起编译的其他文件可见）

### 属性
* 声明和变量一样
* 属性：字段 + 访问器
* var 可写属性： 一个字段 + 一个getter + 一个 setter
* val 只读属性：一个字段 + 一个getter
* 若名字以is开头，getter不增加前缀，setter会将is替换成set


### 构造函数
* 构造方法的参数可以指定默认值
* 使用init语句块进行初始化操作
	* 可以有多个init语句块，会按类中顺序从上到下依次执行
```kotlin
class Bird(val weight: Double = 0.00,
		   val age: Int = 0, val color: String = "blue")

val bird1 = Bird(color = "black")
```

#### 主从构造
* 主构造方法：在类外部定义，只能有一个
* 从构造方法：由constructor定义，可以有多个
	* 对其他构造方法的委托，通过this来调用
	* 由花括号包裹的代码块
```kotlin
class Bird(age: Int) {
	val age: Int

	init {
		this.age = age
	}

	constructor(timestamp: Long): this(DateTime(timestamp)) {
		...
	}
}
```

### 延迟初始化
#### by lazy
* 只能用于val变量
* 首次调用时，才会进行赋值操作
* 系统会加上同步锁，是线程安全的
```kotlin
class Bird(val weight: Double = 0.00,
		   val age: Int = 0, val color: String = "blue") { 
    val sex: String by lazy { 
		if (color == "yellow") "male" else "female"
	}
}
```

#### lateinit
* 主要用于var变量
* 不能用于基本数据类型
```kotlin
class Bird(val weight: Double = 0.00,
		   val age: Int = 0, val color: String = "blue") { 
    lateinit var sex: String

    fun printSex() { 
		this.sex = if (this.color == "yellow") "male" else "female"
		println(this.sex)
	}
}
```

### 继承
* 使用 ":" 表示类的继承和接口实现
* 类和方法默认是不可继承和重写的，需要加上open 

### 多继承
#### 接口模拟实现
```kotlin
interface Flyer { 
	fun fly()
	fun kind() = "..."
	}
interface Animal { 
	val name: String
	fun eat()
	fun kind() = "..."
	}

class Bird(override val name: String) : Flyer, Animal {
	override fun eat() { ... }
	override fun fly() { ... }

	override fun kind() = super<Flyer>.kind()
}
```

#### 内部类模拟实现
* 内部类：包含对外部类实例的引用
* 嵌套类：不包含对外部类实例的引用
```kotlin
open class Horse {
	fun runFast() { ... }
}
open class Donkey {
	fun doLongTimeThing() { ... }
}

clas Mule {
	fun runFast() { 
		HorseC().runFast()
	}
	fun doLongTimeThing() { 
		DonkeyC().doLongTimeThing()
	}

	private inner class HorseC : Horse()
	private inner class DonkeyC : Donkey()
}
```

#### 委托代替实现
* 使用by关键字实现
* 使用具体的类来实现方法逻辑
* 具体调用时可以直接调用委托对象的方法
```kotlin
interface CanFly {
	fun fly()
}
interface CanEat {
	fun eat()
}

open class Flyer : CanFly {
	override fun fly() { ... }
}
open class Animal : CanEat {
	override fun eat() { ... }
}

class Bird(flyer: Flyer, animal: Animal) : 
		CanFly by flyer, CanEat by animal {}

val flyer = Flyer()
val animal = Animal()
val b = Bird(flyer, animal)
b.fly()
b.eat()
```


### 数据类
#### 创建
* 有一个包含了参数的构造方法
* 构造方法的参数要使用var, val进行声明
```kotlin
data class Bird(var weight: Double, 
				var age: Int, 
				var color: String)
```

#### copy方法
* 提供了一个简洁的方式来复制一个对象
* 是一种浅拷贝的方式

#### 解构
* 基于componentN函数
* 默认会根据主构造函数的参数来生成
```kotlin
val b = Bird(20.0, 1, "blue")
val (weight, age, color) = b
```

### object
#### 伴生对象
* 伴随某个类的对象，属于该类
* 全局只有一个实例
* 在类被装载时会被初始化
* 可以用于实现工厂方法模式
```kotlin
class Prize(...) {
	companion object {
		fun isRedpack(prize: Prize): Boolean {}
	}
}

class Prize private constructor(...) {
	companion object {
		fun newRedpackPrize(...) = Prize(...)
	}
}
```

#### 单例
* 使用object实现
* 可以实现接口，可以继承类
```kotlin
object DatabaseConfig {
	var host: String = "localhost"
	var port: Int = 5000
}
```

#### 表达式
* 用于替换Java中的匿名内部类，当其有多个方法时
```kotlin
val comparator = object : Comparator<String> { 
	override fun compare(s1: String?, s2: String?): Int { ... }
}
```



## 类型系统

### 可空类型
* 在任何类型后面加上"?"
	* Int? = Int or null
* 基本类型的可空版本会使用该类型的包装形式
* 使用安全调用”?.“，保证非空时才进行调用
* 使用非空断言调用"!!."，保证在空时会抛出异常
* 使用Elvis操作符”?:“，保证空时有默认值

```kotlin
na?.length  //返回null
na!!.length // 返回KotlinNullPointerException

y = x?:0  //等价于下面语句
val y = if(x !== null) x else 0  
```

### 智能转换
* 使用 is 检查后，编译器会自动进行类型转换
* 条件：要保证变量检查后不会再改变
```kotlin
val stu: Any = Student(Glasses(189))
if(stu is Student) println(stu.glasses)
```

### 强制转换
* 使用 as 操作符, 若对象为空，则抛出异常
* as? 为类型安全版本，若对象为空，则返回null
```kotlin
var stu: Student? = getStu() as Student?
```

### 根类型Any
* Any 是非空类型的根类型
* Any 运行时会自动映射成Object
* Any? 是所有类型的根类型

#### 继承vs子类型化
* 继承属于一种 实现上的复用
* 子类型化属于一种 类型语义的关系，与实现无关


### Nothing
* 是类型层次中最底层的类型
* 是没有实例的类型
* 该类型的表达式不会产生任何值
* 该类型只能包含一个值：null


## 扩展

### 多态
* 子类型多态：子类继承父类
* 参数多态：泛型
* 特设多态：扩展、操作符重载
	* 理解：一个多态函数有多种不同的实现，依赖于其实参而调用相应版本的函数

### 操作符重载
* 使用operator关键字
	* 作用：将一个函数标记为重载一个操作符
* 函数名是Kotlin规定的

```kotlin
data class Area(val value: Double)

operator fun Area.plus(that: Area): Area {
	return Area(this.value + that.value)
}

fun main() {
	Area(1.0) + Area(2.0)
}
```

#### 重载函数名
| 表示式   |  函数 |  
| :--- | :---:|
| a + b |  a.plus(b)  |
| a - b | a.minus(b) |
| a * b | a.times(b) |
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


### 实现
#### 扩展函数
* 使用\<Type\>关键字
* 需要指定一个 **接收者类型**
* 函数中使用this来表示**接收者类型的对象**
* 底层实现：Java中的静态方法

```kotlin
fun MutableList<Int>.exchange(fronIndex: Int，toIndex: Int) {
	...
}
```

#### 扩展属性
```kotlin
val MutableList<Int>.sumIsEven: Boolean
	get() = this.sum() % 2 == 0

//使用
val list = mutableListOf(2,2,4)
list.sumIsEven
```

#### 特殊情况
* 若要声明一个静态的扩展函数，必须要定义在伴生对象上
* 函数名相同时，成员方法优先级高于扩展函数


### 标准库扩展函数
#### apply
* 返回操作对象

##### 定义
```kotlin
fun <T> T.apply(block: T.() -> Unit): T {
	block()
	return this
}
```

##### 例子
```kotlin
bean.apply { 
	titleTV.text = this.title 
}
```

#### let
* 返回运算结果

##### 定义
```kotlin
fun <T,R> T.let(block: (T) -> R): R {
	return block(this)
}
```

##### 例子
```kotlin
val result = student?.let { 
	println(this.age)
	it.age
}
```

#### run
* 返回运算结果

##### 定义
```kotlin
fun <T,R> T.let(block: (T) -> R): R {
	return block()
}
```

##### 例子
```kotlin
run {
	if(!isLogin) loginDialog else getNewAccountDialog
}.show()
```

#### also
##### 定义
```kotlin
fun <T> T.apply(block: T.() -> Unit): T {
	block(this)
	return this
}
```

##### 例子
```kotlin
val result = student?.let { stu ->
	println(stu.age)
	stu.age
}
```

#### takeIf
* 当接收器满足指定条件时，才会执行

##### 定义
```kotlin
fun <T> T.takeIf(predicate: (T) -> Boolean): T? {
	return if(predicate(this)) this else null
}
```

##### 例子
```kotlin
student.takeIf { it.age >= 18 }.let { ... }
```


## 元编程

### 定义
* 程序即数据，数据即程序
* 程序即数据：访问描述程序的数据，即通过反射获取类型信息
* 数据即程序：将数据转化成对应的程序，即代码生成

### 实现方案
* 反射：运行时通过API暴露程序信息
* 动态执行：运行是动态将文本作为代码执行
* 语法糖：通过外部程序（编译器）操作AST实现



## 协程

### 特点
* 协程工作在线程之上
* 协程的切换可以由程序自己来控制

### 接口
* delay 用于挂起协程，不会阻塞线程
* launch 启动一个协程
* runBlocking 启动一个主协程，会阻塞当前线程

```kotlin
fun main() = runBlocking { 
	launch {
		delay(1000L)
		println("")
	}
	println("")
	delay(2000L)
}
```

### 等待
* join() 非阻塞式的等待指定的协程结果
* suspend：该关键字修饰的方法内部还可以调用其他的suspend方法

```kotlin
suspend fun search() {
	delay(1000L)
	println("")
}

fun main() = runBlocking { 
	val job = launch { 
		search()
	}
	println("")
	job.join()
}
```

### 并行
* async 创建一个子协程，该协程并行运行。返回一个Deferred对象
* await() 等待协程运行完成，并获取协程返回的结果

```kotlin
suspend fun searchItemOne(): String {
	delay(1000L)
	return "..."
}
suspend fun searchItemTwo(): String {
	delay(1000L)
	return "..."
}

fun main() = runBlocking { 
	val one = async { searchItemOne() }
	val two = async { searchItemTwo() }
	println("The item is ${one.await()} and ${two.await()}")
}
```



## 用例

### apply方法
```kotlin
//原始代码
val bundle = Bundle()
bundle.putSerializable("url","http://tjjqrby.aiquanbar.cn")
fragment.arguments = bundle


//简化后代码
 fragment.arguments = Bundle().apply { putSerializable("url","http://tjjqrby.aiquanbar.cn") }
```