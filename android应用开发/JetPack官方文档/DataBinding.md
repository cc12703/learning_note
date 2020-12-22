

# DataBinding

* 原始文档： https://developer.android.com/topic/libraries/data-binding


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