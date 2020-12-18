

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


### 使用方法
