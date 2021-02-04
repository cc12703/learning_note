

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


### ViewStubs
* 不像正常视图，ViewStub开始于一个不可见视图
* 当它们变得可见时，或者主动需要inflate时。它们会被其他布局替换掉

#### ViewStubProxy
* 由于ViewStub在视图层次中是不可见的，所以在绑定对象上，该视图也应该是不可见的，已方便被GC
* 由于视图是不可变的，所以在绑定类中VeiwStubProxy会替换ViewStub，用于在ViewStub存在时，存取该对象
* 当inflating其他布局时，一个绑定必须确定连接和新生成的布局
* ViewStubProxy必须监听ViewStub的OnInflateLisener，并确定绑定完成
* 由于指定的时间内只有一个监听器可以存在，所以ViewStubProxy允许设置一个OnInflateLisener，该回调会在绑定确定后被调用

### 立即绑定
* 当变量、可观察对象变更时，绑定会在下一帧显示前被调度来做变更
* 当需要绑定立刻变更时，需要调用executePendingBindings()方法

### 高级绑定
#### 动态变量
* 有时候，绑定类的类型是不需要知道的
    * RecyclerView.Adapter 操作任意布局时，是不需要知道绑定类的类型的
    * 在调用onBindViewHolder()时，都需要指派一个绑定值

##### 例子
* 所有被RecyclerView绑定的布局，都有一个item变量
* BindingHolder对象有一个getBinding()函数用于返回ViewDataBinding对象

```kotlin
override fun onBindViewHolder(holder: BindingHolder, position: Int) {
    item: T = items.get(position)
    holder.binding.setVariable(BR.item, item);
    holder.binding.executePendingBindings();
}
```

### 后台线程
* 你可以在后台线程中修改数据对象，只要数据对象是非数组
* data-binding库会在求值时，对变量、属性进行局部化操作，已避免出现并发问题


### 自定义绑定类名
#### 默认规则
* 大写字母开头
* 移除下划线，大写化后面的字母
* 增加后缀名，Binding
* 类放置于模块包的databinding子包下

#### 自定义
* 使用data元素的class属性进行自定义操作

```kotlin
<!-- 只定义了类名 -->
<data class="ContactItem">
    …
</data>

<!-- 放置于当前包下 -->
<data class=".ContactItem">
    …
</data>

<!-- 使用全限定名，自定义类名和要存放的包 -->
<data class="com.example.ContactItem">
    …
</data>
```


## 绑定适配器
* 用于使用合适的系统接口来设置值
    * 调用setText()来设置值
    * 调用setOnClickListener()来设置事件监听器
* data-binding允许使用自定义的方法来设置值

### 设置属性值
* 当一个绑定值变更时，生成的绑定类必须要使用绑定表达式，在视图上调用setter方法
* 你可以让data-binding库自动选择方法，或者指定一个方法

#### 自动选择方法
* 像属性名example，库会自动查找格式为setExample(arg)，参数又是类型兼容的方法。
* 当进行方法搜索时，不会考虑属性的命名空间，只会考虑属性名和类型

##### 例子
* 对于表达式 android:text="@{user.name}"，库会查找setText(arg)方法，arg类型是user.getName()的返回类型
* 如果uesr.getName()返回字符串类型，库就会查找接收字符串类型的setText()方法
* 如果表达式返回整型，则库会查找接收整型的setText()方法

##### 无属性
* 在给定的名字不存在属性时，data-binding也可以运行。可以通过setter方法来创建对应的属性

```xml
<android.support.v4.widget.DrawerLayout
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:scrimColor="@{@color/scrim}"
    app:drawerListener="@{fragment.drawerListener}">
```
说明
* app:scrimColor对应setScrimColor(int)方法
* app:drawerListener对应setDrawerListener(DrawerListener)方法


#### 自定义方法名
* 对于部分属性名字和setter函数名不一致的情况，可以使用BindingMethods标注来给属性分配一个setter函数
* 该标注用在类上，可以包含多个BindingMethod标注，每个BindingMethod对应一个重命名方法

```kotlin
@BindingMethods(value = [
    BindingMethod(
        type = android.widget.ImageView::class,
        attribute = "android:tint",
        method = "setImageTintList")])
```
说明
* android:tint属性分配了setImageTintList(ColorStateList)方法



#### 提供自定义逻辑
* 一些属性需要自定义绑定逻辑
* 像android:paddingLeft属性，只有setPadding(left, top, right, bottom)方法
* 使用BindingAdapter标注一个静态方法，可以自定义setter逻辑


```kotlin
@BindingAdapter("android:paddingLeft")
fun setPaddingLeft(view: View, padding: Int) {
    view.setPadding(padding,
                view.getPaddingTop(),
                view.getPaddingRight(),
                view.getPaddingBottom())
}
```
说明
* 该例子给paddingLeft属性绑定了函数
* 第一个参数决定了包含该属性的视图的类型
* 第二个参数决定了在表达式中该属性可以接受的类型


##### 多个属性
* 适配器函数可以接受多个属性

```kotlin
@BindingAdapter("imageUrl", "error")
fun loadImage(view: ImageView, url: String, error: Drawable) {
    Picasso.get().load(url).error(error).into(view)
}
```
```xml
<ImageView app:imageUrl="@{venue.imageUrl}" app:error="@{@drawable/venueError}" />
```
说明
* 该函数只有在imageUrl, error属性都被使用，而且数据类型匹配时，才会被调用

##### 任意匹配
* 将reqireAll设置为false时，任意属性被设置时，函数都会被调用

```kotlin
@BindingAdapter(value = ["imageUrl", "placeholder"], requireAll = false)
fun setImageUrl(imageView: ImageView, url: String?, placeHolder: Drawable?) {
    if (url == null) {
        imageView.setImageDrawable(placeholder);
    } else {
        MyImageLoader.loadInto(imageView, url, placeholder);
    }
}
```

##### 获取旧值
* 所有的旧值参数都需要先定义

```kotlin
@BindingAdapter("android:paddingLeft")
fun setPaddingLeft(view: View, oldPadding: Int, newPadding: Int) {
    if (oldPadding != newPadding) {
        view.setPadding(padding,
                    view.getPaddingTop(),
                    view.getPaddingRight(),
                    view.getPaddingBottom())
    }
}
```

##### 事件处理函数
* 只有事件处理函数被定义在接口类、包含抽象方法的抽象类时，才可以被使用

```kotlin
@BindingAdapter("android:onLayoutChange")
fun setOnLayoutChangeListener(
        view: View,
        oldValue: View.OnLayoutChangeListener?,
        newValue: View.OnLayoutChangeListener?
) {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB) {
        if (oldValue != null) {
            view.removeOnLayoutChangeListener(oldValue)
        }
        if (newValue != null) {
            view.addOnLayoutChangeListener(newValue)
        }
    }
}
```
```xml
<View android:onLayoutChange="@{() -> handler.layoutChanged()}"/>
```

##### 多个方法
* 当一个监听器类中有多个方法时，必须要拆成多个监听器类
* 像View.OnAttachStateChangeListener类有两个方法
    * onViewAttachedToWindow(View)
    * onViewDetachedFromWindow(View)
* data-binding库会提供两个接口类，来分别定义两个方法

```kotlin
@TargetApi(Build.VERSION_CODES.HONEYCOMB_MR1)
interface OnViewDetachedFromWindow {
    fun onViewDetachedFromWindow(v: View)
}

@TargetApi(Build.VERSION_CODES.HONEYCOMB_MR1)
interface OnViewAttachedToWindow {
    fun onViewAttachedToWindow(v: View)
}
```

```kotlin
@BindingAdapter(
        "android:onViewDetachedFromWindow",
        "android:onViewAttachedToWindow",
        requireAll = false
)
fun setListener(view: View, detach: OnViewDetachedFromWindow?, attach: OnViewAttachedToWindow?) {
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.HONEYCOMB_MR1) {
        val newListener: View.OnAttachStateChangeListener?
        newListener = if (detach == null && attach == null) {
            null
        } else {
            object : View.OnAttachStateChangeListener {
                override fun onViewAttachedToWindow(v: View) {
                    attach.onViewAttachedToWindow(v)
                }

                override fun onViewDetachedFromWindow(v: View) {
                    detach.onViewDetachedFromWindow(v)
                }
            }
        }

        val oldListener: View.OnAttachStateChangeListener? =
                ListenerUtil.trackListener(view, newListener, R.id.onAttachStateChangeListener)
        if (oldListener != null) {
            view.removeOnAttachStateChangeListener(oldListener)
        }
        if (newListener != null) {
            view.addOnAttachStateChangeListener(newListener)
        }
    }
}
```
说明
* android.databinding.adapters.ListenerUtil 用于跟踪前面设置的监听器，以方便在适配器中移除


### 对象转换
#### 自动对象转换
* 当表达式返回一个Object类型的值时，库会选择一个方法
* Object类型将会被转换成被选择方法的参数的类型

```xml
<TextView
   android:text='@{userMap["lastName"]}'
   android:layout_width="wrap_content"
   android:layout_height="wrap_content" />
```
说明
* userMap返回的值，将会被转换成setText(CharSequence)函数中参数的类型

#### 自定义转换
* 使用BindingConversion标注一个静态方法可以实现

```kotlin
@BindingConversion
fun convertColorToDrawable(color: Int) = ColorDrawable(color)
```
```xml
<View
   android:background="@{isError ? @color/red : @color/white}"
   android:layout_width="wrap_content"
   android:layout_height="wrap_content"/>
```

## 将布局视图绑定到架构组件上

### LiveData
* 使用LiveData作为数据绑定源，在数据变更时自动通知界面
* LiveData知道已订阅观察者的生命周期
* 要在绑定类中使用LiveData，则需要设置生命周期的拥有者

```kotlin
class ViewModelActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        val binding: UserBinding = DataBindingUtil.setContentView(this, R.layout.user)

        // 设置当前activity为生命周期拥有者
        binding.setLifecycleOwner(this)
    }
}
```

### ViewModel
* 可以使用ViewModel来管理UI相关的数据
* data-binding库可以和ViewModel组件无缝工作
* 使用ViewModel可以将布局中的UI逻辑移入到该组件中，以方便测试


#### 创建
```kotlin
class ScheduleViewModel : ViewModel() {
    val userName: LiveData

    init {
        val result = Repository.userName
        userName = Transformations.map(result) { result -> result.value }
    }
}
```

#### 实例化
```kotlin
class ViewModelActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        // 获取viewModel实例
        val userModel: UserModel by viewModels()

        val binding: UserBinding = DataBindingUtil.setContentView(this, R.layout.user)

        // 给绑定类设置viewModel
        binding.viewmodel = userModel
    }
}
```

#### 使用
```xml
<CheckBox
    android:id="@+id/rememberMeCheckBox"
    android:checked="@{viewmodel.rememberMe}"
    android:onCheckedChanged="@{() -> viewmodel.rememberMeChanged()}" />
```

### 可观测ViewModel
* ViewModel实现Observable后，可以通知其他应用组件数据变更
* 使用可观测ViewModel可以给你更多的控制权


```kotlin
open class ObservableViewModel : ViewModel(), Observable {
    private val callbacks: PropertyChangeRegistry = PropertyChangeRegistry()

    override fun addOnPropertyChangedCallback(
            callback: Observable.OnPropertyChangedCallback) {
        callbacks.add(callback)
    }

    override fun removeOnPropertyChangedCallback(
            callback: Observable.OnPropertyChangedCallback) {
        callbacks.remove(callback)
    }

    //通知观察者所有属性都变更
    fun notifyChange() {
        callbacks.notifyCallbacks(this, 0, null)
    }

    
     //通知观察者指定属性变更
     // fieldId 参数是属性对应的BR标识，该属性被@Bindable标注
    fun notifyPropertyChanged(fieldId: Int) {
        callbacks.notifyCallbacks(this, fieldId, null)
    }
}
```


## 双向数据绑定
* 使用双向数据绑定，你可以给属性设置值，同时给属性设置监听器以响应数据变更
* @={}符号包含了 "=" 记号，可以接收对属性的数据变更，同时监听用户的更新操作
* 为了响应后端数据的变更，需要变量实现可观测性

### 使用
```xml
<CheckBox
    android:id="@+id/rememberMeCheckBox"
    android:checked="@={viewmodel.rememberMe}"
/>
```

### 定义数据
```kotlin
class LoginViewModel : BaseObservable {
    // val data = ...

    @Bindable
    fun getRememberMe(): Boolean {
        return data.rememberMe
    }

    fun setRememberMe(value: Boolean) {
        // 避免无限循环
        if (data.rememberMe != value) {
            data.rememberMe = value

            //响应变更
            saveData()

            // 通知观察者，数据已变更
            notifyPropertyChanged(BR.remember_me)
        }
    }
}
```

### 自定义属性
* 需要使用@InverseBindingAdapter，@InverseBindingMethod 标注

#### 例子
* 给MyView的time属性创建双向绑定
* 操作步骤
    1. 给setter函数标注@BindingAdapter
    2. 给getter函数标注@InverseBindingAdapter
    3. 创建设置监听器函数，并标注@BindingAdapter


```kotlin
@BindingAdapter("time")
@JvmStatic fun setTime(view: MyView, newValue: Time) {
    // 很重要，防止无限循环
    if (view.time != newValue) {
        view.time = newValue
    }
}

@InverseBindingAdapter("time")
@JvmStatic fun getTime(view: MyView) : Time {
    return view.getTime()
}

@BindingAdapter("app:timeAttrChanged")
@JvmStatic fun setListeners(
        view: MyView,
        attrChange: InverseBindingListener
) {
    // 给点击、触摸、获取焦点设置监听器
}
```

### 转换器
* 用于在变量显示前，将变量格式化、翻译、修改
* 对于双向绑定，需要定义一个反向转换器

```xml
<EditText
    android:id="@+id/birth_date"
    android:text="@={Converter.dateToString(viewmodel.birthDate)}"
/>
```
```kotlin
object Converter {
    //定义反向转换
    @InverseMethod("stringToDate")
    @JvmStatic fun dateToString(
        view: EditText, oldValue: Long,
        value: Long
    ): String {
        // 将长整型转换成字符串
    }

    @JvmStatic fun stringToDate(
        view: EditText, oldValue: String,
        value: String
    ): Long {
        // 将字符串转换成长整型
    }
}
```

### 无限循环
* 在使用双向绑定时，要小心不要引入无限循环

#### 原因
1. 用户改变属性，标注@InverseBindingAdapter的函数被调用
1. 新的值被赋给属性， 标注@BindingAdapter的函数被调用
1. 函数有可能再触发标注@InverseBindingAdapter的函数被调用

#### 措施
* 在标注@BindingAdapter的函数中，增加新值和旧值的比较操作


### 双向绑定属性
| 类 | 属性 | 绑定适配器 |
| -- | -- | -- |
| AdapterView | android:selection | AdapterViewBindingAdapter |
| TextView | android:text | TextViewBindingAdapter |
| TabHost | android:currentTab | TabHostBindingAdapter |
| SeekBar | android:progress | SeekBarBindingAdapter |