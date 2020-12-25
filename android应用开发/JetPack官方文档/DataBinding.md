

# DataBinding

* 原始文档： https://developer.android.com/topic/libraries/data-binding

[TOC]


## 概述
### 总体
* data-binding允许在布局文件中直接绑定UI控件和数据源
* 使用声明式语法，而不用任何编程

#### 未使用data-binding
```kotlin
findViewById<TextView>(R.id.sample_text).apply {
    text = viewModel.userName
}
```

#### 使用data-binding
```xml
<TextView
    android:text="@{viewmodel.userName}" />
```

### 优点
* 让你在activitiy中不用写许多UI框架的调用
* 让你的代码更简单，更容易维护
* 提高应用的性能，防止内存泄漏和空指针异常


### 使用
#### 布局和绑定表达式
* 表达式语言允许你写表达式来连接变量到布局中的控件
* data-binding库会自动生成类，来用数据对象绑定布局中的控件
* data-binding库提供了imports, variables, includes等特性
* 这些特性都是可以和已存在的布局无缝共存的

```xml
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:app="http://schemas.android.com/apk/res-auto">
    <data>
        <variable
            name="viewmodel"
            type="com.myapp.data.ViewModel" />
    </data>
    <ConstraintLayout... /> <!--布局的根元素 -->
</layout>
```

说明
* 变量绑定在data元素中定义
* 所有的元素都被包裹在layout标签


#### 监控数据变量
* data-binding库提供了相关的类和方法，可以更容易的监控数据的变动
* 当底层数据改变时，你不用担心是否需要刷新界面。你可以让你的变量、属性变成可观测的
* data-binding允许你创建对象，字段，数组的可观测对象

#### 绑定适配器
* 对于每个布局表达式，都会有一个绑定适配器用于调用系统接口来设置对应的属性和监听器
* 举例，适配器可以调用setText()来设置文本属性，可以调用setOnClickListener()来给点击事件添加监控器
* 大部分公共适配器的用法都可以在这里找到
* 所有适配器都在android.databinding.adapters包下

```kotlin
@BindingAdapter("app:goneUnless")
fun goneUnless(view: View, visible: Boolean) {
    view.visibility = if (visible) View.VISIBLE else View.GONE
}
```

#### 双向数据绑定
* data-binding库支持双向数据绑定
* 该功能可以同时支持以下两种操作
    * 接收属性上的数据变动并更新到控件上
    * 监听用户对控件的操作并更新到属性上


## 开始
### 环境
* 需要从support仓库中下载该库
* 在build.gradle文件中启用该功能

```groovy
android {
    ...
    buildFeatures {
        dataBinding true
    }
}
```

### IDE的支持
* android Studio对于data-binding代码支持需要编辑特性
* 例子
    * 语法高亮
    * 标记出表达式语句的语法错误
    * xml的代码补全
    * 参考资料
* 布局编辑器的预览界面会显示绑定表达式的默认值



## 绑定表达式
### 概述
* 表达式语法允许你写表达式来处理控件分发过来的事件
* data-binding布局文件包括以下解部分
    * layout标记作为根标记
    * daa标记用于定义变量
    * 控件的根元素

#### 例子
```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"/>
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.lastName}"/>
   </LinearLayout>
</layout>
```

说明
* data区定义了user变量，可以在该布局文件中使用
* 使用 **\@{}** 语法用于访问变量中的属性

### 数据对象
* 设想一个普通的旧式类 User
* 在应用中这种情况比较普遍，保存的数据只读取一次后就不再改变
* 这种类也会带有一些约定的方法，用于读取、设置数据
* 从data-binding来看，这两种类都是一样的
* 表达式 @{user.firstName} 用于读取给定类的firstName属性，也会调用getFirstName()函数，或者firstName()函数


```java
public class User {
  public final String firstName;
  public final String lastName;
  public User(String firstName, String lastName) {
      this.firstName = firstName;
      this.lastName = lastName;
  }
}

public class User {
  private final String firstName;
  private final String lastName;
  public User(String firstName, String lastName) {
      this.firstName = firstName;
      this.lastName = lastName;
  }
  public String getFirstName() {
      return this.firstName;
  }
  public String getLastName() {
      return this.lastName;
  }
}
```

### 绑定数据
* 每个布局文件都会生成一个绑定类
* 默认情况下，类名是就与布局文件名生成的：转换文件名为大小写形式，并加上绑定后缀名
    * 例子：activity_main.xml 转换后 ActivityMainBinding
* 该类保存了所有布局属性到控件的绑定，知道如何给绑定表达式赋值

#### 创建绑定对象
##### 方式1
```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    val binding: ActivityMainBinding = 
            DataBindingUtil.setContentView(this, R.layout.activity_main)
    binding.user = User("Test", "User")
}
```

##### 方式2
```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    val binding: ActivityMainBinding = 
            ActivityMainBinding.inflate(getLayoutInflater())
    binding.user = User("Test", "User")
}
```

##### 方式3
* 用于在Fragment, ListView, RecyclerView适配器中绑定项目

```kotlin
val listItemBinding = ListItemBinding.inflate(layoutInflater, viewGroup, false)
// or
val listItemBinding = DataBindingUtil.inflate(layoutInflater, R.layout.list_item, viewGroup, false)
```

### 表示式语法
#### 公共特性
* 表达式语法看起来非常像受控代码中的表达式
* 使用以下的操作符合关键字
    * 数学操作：+, -, /, *, %
    * 字符串连接：+
    * 逻辑操作：&&, ||
    * 二进制操作：&, |, ^
    * 一元操作：+， -， !， ~
    * 移位操作：>>, >>>, <<
    * 比较： ==，>, <, >=, <=
    * instanceof
    * 分组：()
    * 字面量：字母、字符、数字、null
    * 数据类型转换
    * 方法调用
    * 属性访问
    * 数组访问
    * 三元操作：?:

#### 缺失操作符
* this
* super
* new
* 显示的通用调用

#### null聚合操作符
* ?? 含义：如果左操作符不为空则选该操作符，若为空则选择右操作符

##### 例子
```xml
android:text="@{user.displayName ?? user.lastName}"

<!-- 等价于 -->
android:text="@{user.displayName != null ? user.displayName : user.lastName}"
```

#### 属性引用
* 使用以下格式，表达式可以引用一个类的属性
* 不管属性是纯的字段、get函数、ObservableField对象

```xml
android:text="@{user.lastName}"
```

#### 避免空指针异常
* 生成的绑定代码会自动检查空值，避免空指针异常
* 对于表达式 @{user.name}，如果user是null，则user.name将会赋予默认值null
* 如果你引用 user.age 且age是整型，则默认值为0

#### 控件引用
* 表达式可以引用同一布局中的其他控件，通过使用标识符
* 标识符将会转换成 驼峰式大小写

```xml
<EditText
    android:id="@+id/example_text"
    android:layout_height="wrap_content"
    android:layout_width="match_parent"/>
<TextView
    android:id="@+id/example_output"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@{exampleText.text}"/>
```

#### 数组
* 通用数组可以通过 [ ] 操作符来读取
* 包括：arrays, lists, sparse lists, maps

```xml
<data>
    <import type="android.util.SparseArray"/>
    <import type="java.util.Map"/>
    <import type="java.util.List"/>
    <variable name="list" type="List&lt;String>"/>
    <variable name="sparse" type="SparseArray&lt;String>"/>
    <variable name="map" type="Map&lt;String, String>"/>
    <variable name="index" type="int"/>
    <variable name="key" type="String"/>
</data>

android:text="@{list[index]}"

android:text="@{sparse[index]}"

android:text="@{map[key]}"

android:text="@{map.key}"
```

#### 字面量
* 如果属性值用单引号包围，则字面量用双引导
* 如果属性值用双引号包围，则字面量可以用反引导

```xml
android:text='@{map["firstName"]}'
android:text="@{map[`firstName`]}"
```

#### 资源
* 可以直接引用应用的资源
* 可以求值格式化字符串
* 可以将属性引用和控件引用作为资源参数

```xml
android:padding="@{large? @dimen/largePadding : @dimen/smallPadding}"

android:text="@{@string/nameFormat(firstName, lastName)}"
android:text="@{@plurals/banana(bananaCount)}"

android:text="@{@string/example_resource(user.lastName, exampleText.text)}"

```

##### 资源类型
| 类型 | 正常引用 | 表达式引用 |
| -- | -- | -- | 
| String[] | @array | @stringArray |
| int[] | @array | @intArray |
| TypedArray | @array | @typedArray |
| Animator | @animator | @animator | 
| StateListAnimator | @animator | @stateListAnimator |
| color int | @color | @color |
| ColorStateList | @color | @colorStateList |


#### 事件处理
* data-binding库允许你写表达式来处理控件分发过来的事件
* 事件名字由监听器的方法名确定，除了一些特例
    * 例子：View.OnClickListener包含onClick()函数，对应事件为android:onClick


##### 特殊事件
| 类 | 监听器方法 | 事件属性 |
| -- | -- | -- | 
| SearchView | setOnSearchClickListener | android:onSeachClick | 
| ZoomControls | setOnZoomInClickListener | android:onZoomIn | 
| ZoomControls | setOnZoomOutClickListener | android:onZoomOut |

##### 处理方式
###### 方法引用
* 可以遵照监听器方法的签名来引用方法
* data-binding库会在一个监听器中包含引用的方法和方法所有者的对象，并给目标控件设置该监听器
* 如果表达式求值结果为null，则库不会创建监听器，并设置null给目标控件

###### 监听器绑定
* lambda表达式在事件发生时才会求值
* data-binding库总是会创建一个监听器并设置给目标控件
* 当事件被分发时，监听器将会求值lambda表达式


##### 方法引用
* 事件可以直接绑定给处理方法，非常像android:onClick在activityh中被指派一个方法
* 相对于View的onClick属性，方法引用的主要优势就是表达式是在编译时被处理的
    * 如果方法不存在或者签名不正确，可以在编译阶段就得到信息
* 相比于监听器绑定最大的不同在于实际监听器创建是在数据被绑定时，而不是事件被触发时
    * 如果你希望事件发生时表示式被求值，请使用监听器绑定
* 給事件分配一个处理器，可以使用正常的绑定表达式

```kotlin
class MyHandlers {
    fun onClickFriend(view: View) { ... }
}
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android">
   <data>
       <variable name="handlers" type="com.example.MyHandlers"/>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <TextView android:layout_width="wrap_content"
           android:layout_height="wrap_content"
           android:text="@{user.firstName}"
           android:onClick="@{handlers::onClickFriend}"/>
   </LinearLayout>
</layout>
```

##### 监听器绑定
* 绑定的表达式，在事件发生时才会执行
* 这个和方法引用很相似，但是可以运行任意数据绑定表达式
* 在方法引用中，方法的参数必须匹配事件监听器方法的参数
* 在监听器绑定中，只需要返回值匹配事件监听器方法的返回值即可

###### 回调函数
* 当在表达式中使用回调函数时，data-binding会创建一个监听器对象并注册给该事件
* 当控件触发该事件时，data-binding会求值指定的表达式
* 在正常的绑定表达式中，当前表达式被求值时，你仍然会获得null和线程安全

###### 监听器参数
* 监听器绑定提供了两种方式用于处理参数
    * 忽略所有的参数
    * 命名所有的参数

###### 例子
```xml
android:onClick="@{() -> presenter.onSaveClick(task)}"
android:onClick="@{(view) -> presenter.onSaveClick(task)}"

android:onClick="@{(theView) -> presenter.onSaveClick(theView, task)}"
```

###### 默认值
* 如果表达式因为null而无法被求值，则data-binding会返回对应类型的默认值
* 例子：引用类型为null， 整数类型为0，布尔类型为false

###### 谓语
* 如果需要在表达式中使用谓语(三元语法)，可以使用void作为符号

```xml
android:onClick="@{(v) -> v.isVisible() ? doSomething() : void}"
```

###### 避免复杂监听器
* 监听器表达式非常强大，可以使你的代码更容易阅读
* 另一面，如果使用太复杂表达式，会使你的布局更难阅读和维护
* 表达式应该足够简单：只从控件中传递数据给回调函数，在回调函数中实现业务逻辑


#### 其他特性
* 导入：更容易在布局文件中引用类
* 变量：允许你描述一个属性，用于绑定表达式
* include：在应用中重用复杂的布局

##### 导入
* 允许在布局文件中更容易的引用一个类
* 在data元素内可以使用0、多个import元素
* java.lang.* 会被自动导入

```xml
<data>
    <import type="android.view.View"/>
</data>

<TextView
   android:text="@{user.lastName}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"
   android:visibility="@{user.isAdult ? View.VISIBLE : View.GONE}"/>
```

###### 类型别名
* 当类名冲突时，可以将一个类名重命名成别名

```xml
<import type="android.view.View"/>
<import type="com.example.real.estate.View"
        alias="Vista"/>
```

###### 导入其他类
* 导入可以用于类型引用
* 导入可以用于表达式的case部分
* 导入可以用于静态属性、方法的引用

```xml
<data>
    <import type="com.example.User"/>
    <import type="java.util.List"/>
    <variable name="user" type="User"/>
    <variable name="userList" type="List&lt;User>"/>
</data>

<TextView
   android:text="@{((User)(user.connection)).lastName}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

```xml
<data>
    <import type="com.example.MyStringUtils"/>
    <variable name="user" type="com.example.User"/>
</data>
…
<TextView
   android:text="@{MyStringUtils.capitalize(user.lastName)}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

##### 变量
* 在data元素中可以使用多个variable元素
* 每个variable元素描述一个属性：可以被设置，可以在绑定表达式中本地使用

```xml
<data>
    <import type="android.graphics.drawable.Drawable"/>
    <variable name="user" type="com.example.User"/>
    <variable name="image" type="Drawable"/>
    <variable name="note" type="String"/>
</data>
```

###### 类型
* 变量类型是在编译时被检查处理的
* 如果变量实现了Observable接口或者是Observale集合，则变量将会被观察
* 如果变量只是一个基础类或者没有实现Observable接口，则变量将不会被观察

###### 合并
* 如果给不同配置（竖屏、横屏）创建不同的布局文件，则变量会被合并
* 在这些布局文件中定义变量时需要保证名字不冲突

###### 默认值
* 生成的类中会给每个变量生成一个setter函数、一个getter函数
* 变量在没有被设置时，会有一个默认值
    * 引用类型为null
    * 整数类型为0
    * 布尔类型为false

###### context变量
* 该变量会被自动生成，可以用于表达式中
* 该变量值来源于根控件的getContext()方法
* 该变量可以通过定义同名的变量被覆盖掉

##### include
* include其他布局文件时，可以传入变量
* 不支持在merge元素中直接使用include

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:bind="http://schemas.android.com/apk/res-auto">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <LinearLayout
       android:orientation="vertical"
       android:layout_width="match_parent"
       android:layout_height="match_parent">
       <include layout="@layout/name"
           bind:user="@{user}"/>
       <include layout="@layout/contact"
           bind:user="@{user}"/>
   </LinearLayout>
</layout>
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
        xmlns:bind="http://schemas.android.com/apk/res-auto">
   <data>
       <variable name="user" type="com.example.User"/>
   </data>
   <merge><!-- Doesn't work -->
       <include layout="@layout/name"
           bind:user="@{user}"/>
       <include layout="@layout/contact"
           bind:user="@{user}"/>
   </merge>
</layout>
```

## 可观察数据对象
* 可观察性是指一个对象在其内部数据变更时会通知其他对象
* data-binding库允许对象、属性、集合变成可观察的
* 一个普通对象也可以用于data-binding，但是无法在修改对象后自动触发界面更新
* data-binding库可以用来给数据对象增加这种能力，有点像监听器

### 可观察属性
* 可观察性工作涉及创建类实现Observale接口
* 如果只有几个少量的属性，这些工作会变得不值得
* 这种情况下可以使用通用的Observable类，使基本数据类型具有可观察性

#### 类
* ObservableBoolean
* ObservableByte
* ObservableChar
* ObservableShort
* ObservableInt
* ObservableLong
* ObservableFloat
* ObservableDouble
* ObservableParcelable

#### 特点
* 一个自包含的可观察对象，内部只有一个属性
* 基础类型版本可以避免在存取操作时发生拆箱、装箱操作

#### 例子
```kotlin
class User {
    val firstName = ObservableField<String>()
    val lastName = ObservableField<String>()
    val age = ObservableInt()
}

//使用
user.firstName = "Google"
val age = user.age
```

### 可观察集合
* 一些应用会使用动态结构保存数据
* 可观察集合运行使用键来存取这些结构

#### ObservableArrayMap
* 键可以使用引用类型，像String

##### 例子
```kotlin
ObservableArrayMap<String, Any>().apply {
    put("firstName", "Google")
    put("lastName", "Inc.")
    put("age", 17)
}
```

```xml
<data>
    <import type="android.databinding.ObservableMap"/>
    <variable name="user" type="ObservableMap<String, Object>"/>
</data>
…
<TextView
    android:text="@{user.lastName}"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
<TextView
    android:text="@{String.valueOf(1 + (Integer)user.age)}"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
```

#### ObservableArrayList
* 键是整型的

##### 例子
```kotlin
ObservableArrayList<Any>().apply {
    add("Google")
    add("Inc.")
    add(17)
}
```

```xml
<data>
    <import type="android.databinding.ObservableList"/>
    <import type="com.example.my.app.Fields"/>
    <variable name="user" type="ObservableList<Object>"/>
</data>
…
<TextView
    android:text='@{user[Fields.LAST_NAME]}'
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
<TextView
    android:text='@{String.valueOf(1 + (Integer)user[Fields.AGE])}'
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
```

### 可观察对象
* 该对象实现了Observable接口
* 该接口允许注册一些监听者，希望在对象属性变动时被通知到

#### BaseObservable
* Observable接口有一套添加、删除监听器的机制，但是需要实现者决定何时发送通知
* 为了简化开发，data-binding库提供了BaseObservable类
* 该类实现了监听者的注册机制
* 通过使用Bindable标注和调用notifyPropertyChanged()来实现可观察的功能

##### 例子
```kotlin
class User : BaseObservable() {

    @get:Bindable
    var firstName: String = ""
        set(value) {
            field = value
            notifyPropertyChanged(BR.firstName)
        }

    @get:Bindable
    var lastName: String = ""
        set(value) {
            field = value
            notifyPropertyChanged(BR.lastName)
        }
}
```

##### 生成
* data-binding库会在模块包中生成一个名为BR的类
* 该类包含了用于数据绑定的资源ID号
* Bindable标注会在BR类中生成一个数据项

#### 其他方法
* 如果数据类的基类无法修改，则可以使用PropertyChangeRegistry来实现可观察性


## 生成的绑定类
* 绑定类用于存取布局中的变量和控件
* 绑定类链接着布局变量和布局中的控件
* 绑定类的包名和类名都可以自定义
* 所有的绑定类都继承于ViewDataBinding类

### 名字
* 每个布局文件会生成一个对应的绑定类
* 类名字基于布局文件名生成：将布局文件名转成骆驼大小写形式，加上Binding后缀
    * 例子：activity_main.xml 对应类名 ActivityMainBinding

### 创建绑定对象
* 绑定对象会在布局inflating后立刻生成
* 这样可以确保在使用布局中的表达式绑定视图控件之前，视图层次不会发生变动
* 大部分情况下，都可以使用绑定类中的静态

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    val binding: MyLayoutBinding = MyLayoutBinding.inflate(layoutInflater)

    val binding: MyLayoutBinding = MyLayoutBinding.inflate(getLayoutInflater(), viewGroup, false)

    //布局已经生成，可以直接绑定
    val binding: MyLayoutBinding = MyLayoutBinding.bind(viewRoot)

    //绑定类型未知
    val viewRoot = LayoutInflater.from(this).inflate(layoutId, parent, attachToParent)
    val binding: ViewDataBinding? = DataBindingUtil.bind(viewRoot)

    setContentView(binding.root)
}
```

### 带ID的视图控件
* data-binding会在绑定类中给每个有ID的视图控件创建一个不可变的属性
* 库会中视图层次中提取这些带ID的控件，效率会比调用findViewById()更高


### 变量
* data-binding会给布局中定义的变量生成一个存取方法（包括setter, getter）

```xml
<data>
   <import type="android.graphics.drawable.Drawable"/>
   <variable name="user" type="com.example.User"/>
   <variable name="image" type="Drawable"/>
   <variable name="note" type="String"/>
</data>
```


### ViewStub