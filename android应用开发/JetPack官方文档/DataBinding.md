

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