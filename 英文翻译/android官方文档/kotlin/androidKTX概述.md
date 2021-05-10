

# androidKTX概述

* https://developer.android.com/kotlin/ktx

[TOC]



## 概述
* android KTX是一个Kotlin扩展函数集，被用于Jetpack和其他android库中
* KTX扩展提供了简洁的、符合Kotlin使用习惯的Jetpack、android平台、其他库的API
* 使用了以下的Kotlin特性
    * 扩展函数
    * 扩展属性
    * Lambda
    * 命名参数
    * 参数默认值
    * 协程


### 举例
* 当使用SharePreferences时，在修改数据前需要创建一个编辑器对象
    * 当完成编辑时，需要apply或者commit这些变动
* 示例
    ```kotlin
    sharedPreferences
            .edit()  // 创建一个编辑器
            .putBoolean("key", value)
            .apply() // 异步将数据写入磁盘
    ```
* Kotlin的lambda非常适合这种场景
    * 在编辑器被创建后，它允许你使用更简洁的方法将代码块传入执行
    * 它让代码执行，并让SharePreferences原子的应用这些变动


#### SharePreferences.edit
* SharePreferences.edit() 是KTX的核心函数之一
    * 该函数接收一个可选的布尔类型作为首个参数，来表明是commit还是apply变动
    * 该函数以lambda的形式接收一个需要执行的动作
* 示例代码
    ```kotlin
    // SharedPreferences.edit 一个扩展函数
    // inline fun SharedPreferences.edit(
    //         commit: Boolean = false,
    //         action: SharedPreferences.Editor.() -> Unit)

    // 异步提交一个新值
    sharedPreferences.edit { putBoolean("key", value) }

    // 同步提交一个新值
    sharedPreferences.edit(commit = true) { putBoolean("key", value) }
    ```
* 调用者可以选择是commit还是apply变动
* action lambda作为SharedPreferences.Editor的一个匿名扩展函数，就像函数签名中的那样返回值为Unit
    * 这就是为什么在代码块中，你可以直接调用SharedPreferences.Editor的功能
* SharedPreferences.edit()签名中包含一个inline关键字
    * 该关键字用于告诉Kotlin编译器，每次函数被使用时，需要拷贝粘贴该函数编译后的字节码
    * 这避免了每次函数被调用时，都要实例化一个action类的开销


### 总结
* KTX库提供的用于增强的手段包括
    * 使用lambda传递代码
    * 使用可以被覆盖的默认值
    * 使用inline扩展函数给已存在API增加功能




## 在工程中使用KTX

### 加入依赖
build.gradle文件
```groovy
repositories {
    google()
}
```


## AndroidX模块

### 概述
* KTX被组织成多个模块，每个模块包含一个、多个包
* 你必须在build.gradle中给每个模块制品加入对应的依赖信息
    * 记住要给每个制品追加版本号
* KTX包含一个单独的核心模块，用于给框架API提供扩展、许多领域相关的扩展
* 除了核心模块，所有的KTX模块制品都对应一个Java的依赖库
    * 替换规则：将androidx.fragment:fragment替换成androidx.fragment:fragment-ktx


### 核心KTX
* 核心KTX模块提供了anroid框架中公共库的扩展
* 这些库没有对应的Java依赖
* build.gradle示例
    ```groovy
    dependencies {
        implementation "androidx.core:core-ktx:1.3.2"
    }
    ```
* 模块中包含的包：androidx.core.xxxx