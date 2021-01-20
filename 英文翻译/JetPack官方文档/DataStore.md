

# DataStore

* 原始文档： https://developer.android.com/topic/libraries/architecture/datastore

[TOC]

## 简介
* DataStore是一个数据存储的解决方案
* 允许你存储键-值对、带protocol buffers的类型化对象
* DataStore使用Kotlin协程和Flow来实现异步化、一致性、事务性

## 实现方案
* DataStore提供了两个不同实现：Preferences 和 Proto

### Preferences
* 使用键进行存储和读取数据
* 不需要预先定义模式(schema)
* 不提供类型安全

### Proto
* 将数据作为自定义的类型来存储
* 需要预先定义模式(schema)，使用protocol buffers
* 提供类型安全


## 依赖

### typed DataStore
```groovy
// Typed DataStore (Typed API surface, such as Proto)
dependencies {
  implementation "androidx.datastore:datastore:1.0.0-alpha06"

  // optional - RxJava2 support
  implementation "androidx.datastore:datastore-rxjava2:1.0.0-alpha06"

  // optional - RxJava3 support
  implementation "androidx.datastore:datastore-rxjava3:1.0.0-alpha06"
}

//或者，使用以下组件，不含有android依赖项
dependencies {
  implementation "androidx.datastore:datastore-core:1.0.0-alpha06"
}
```

### preferences DataStore
```groovy
// Preferences DataStore (SharedPreferences like APIs)
dependencies {
  implementation "androidx.datastore:datastore-preferences:1.0.0-alpha06"

  // optional - RxJava2 support
  implementation "androidx.datastore:datastore-preferences-rxjava2:1.0.0-alpha06"

  // optional - RxJava3 support
  implementation "androidx.datastore:datastore-preferences-rxjava3:1.0.0-alpha06"
}

//或者，使用以下组件，不含有android依赖项
dependencies {
  implementation "androidx.datastore:datastore-preferences-core:1.0.0-alpha06"
}
```


## 存储键值对

### 概述
* 库使用了DataStore和Preferences类，实现了简单键值对的持久化储存

### 创建
* 使用 Context.createDataStore()

```kotlin
val dataStore: DataStore<Preferences> 
        = context.createDataStore( name = "settings" )
```

### 读取
* 需要为每个值，使用键类型函数来定义一个对应的键

```kotlin
val EXAMPLE_COUNTER = intPreferencesKey("example_counter")
val exampleCounterFlow: Flow<Int> = dataStore.data
  .map { preferences ->
    // 无类型安全
    preferences[EXAMPLE_COUNTER] ?: 0
}
```

### 写入
* 库提供了edit()来实现事务化的更新数据
* transform参数接收一个代码块，来执行自定义的更新逻辑

```kotlin
suspend fun incrementCounter() {
  dataStore.edit { settings ->
    val currentCounterValue = settings[EXAMPLE_COUNTER] ?: 0
    settings[EXAMPLE_COUNTER] = currentCounterValue + 1
  }
}
```

## 存储类型对象

### 概述
* 库使用了DataStore和protocol buffers来实现了类型对象的持久化

### 定义schema
* 需要在app/src/mian/proto目录下，通过proto文件预定义一个schema
* schema定义了对象的类型

```proto
syntax = "proto3";

option java_package = "com.example.application";
option java_multiple_files = true;

message Settings {
  int32 example_counter = 1;
}
```

### 创建
1. 创建Serializer\<T\>的实现类
    * 类型T就是定义在proto中的类型
    * 串行化类告诉DataStore如何读、写数据
1. 创建DataStore\<T\>实例
    * 使用Context.createDataStore()
    * fileName参数告诉DataStore使用哪个文件存储数据
    * serializer参数告诉DataStore使用哪个串行化类

```kotlin
object SettingsSerializer : Serializer<Settings> {
  override val defaultValue: Settings = Settings.getDefaultInstance()

  override fun readFrom(input: InputStream): Settings {
    try {
      return Settings.parseFrom(input)
    } catch (exception: InvalidProtocolBufferException) {
      throw CorruptionException("Cannot read proto.", exception)
    }
  }

  override fun writeTo(
    t: Settings,
    output: OutputStream) = t.writeTo(output)
}

val settingsDataStore: DataStore<Settings> = context.createDataStore(
  fileName = "settings.pb",
  serializer = SettingsSerializer
)
```

### 读取
* 使用DataStore的data属性，Flow类型

```kotlin
val exampleCounterFlow: Flow<Int> = settingsDataStore.data
  .map { settings ->
    // exampleCounter属性从proto中生成
    settings.exampleCounter
  }
```

### 写入
* 提供updateData()方法来事务性的更新存储的对象
* updateData()会提供给你数据的当前状态
* 在一个原子性的读-写-修改操作中完成数据的更新

```kotlin
suspend fun incrementCounter() {
  settingsDataStore.updateData { currentSettings ->
    currentSettings.toBuilder()
      .setExampleCounter(currentSettings.exampleCounter + 1)
      .build()
    }
}
```


## 使用同步代码
* DataStore的一个优点就是使用异步API
* 但是有的时候修改你正在使用的代码为异步是不太可能的
    * 例子：在一个使用了同步磁盘IO的代码库上工作、依赖库不支持异步API

### 方法
* Kotlin提供了runBlocking()协程生成器，可以使用该生成器从DataStore中同步读取数据
* RxJava在 Flowable 中提供了阻塞方法

```kotlin
val exampleData = runBlocking { dataStore.data.first() }
```

### 其他
* 在UI线程中执行同步IO操作会导致ANR
* 先使用异步模式，从DataStore中预加载数据
    * DataStore会异步读取数据并缓存在内存中
* 之后再用runBlocking()同步读取数据