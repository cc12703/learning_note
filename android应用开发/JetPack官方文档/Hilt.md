
# Hilt

* 原始文档： https://developer.android.com/training/dependency-injection/hilt-android


## 概述
* 是一个依赖注入库，减少手工管理依赖时需要的模板代码
* 提供另一个标准的方式来使用依赖注入
    * 为每个android类提供一个容器
    * 自动管理它们的生命周期
* 构建在Dagger库之上的


## 步骤

### 添加依赖库
```gradle

//root.gradle
buildscript {
    ...
    dependencies {
        ...
        classpath 'com.google.dagger:hilt-android-gradle-plugin:2.28-alpha'
    }
}


//app.gradle
apply plugin: 'kotlin-kapt'
apply plugin: 'dagger.hilt.android.plugin'

android {
    
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
  }
}

dependencies {
    implementation "com.google.dagger:hilt-android:2.28-alpha"
    kapt "com.google.dagger:hilt-android-compiler:2.28-alpha"
}
```

