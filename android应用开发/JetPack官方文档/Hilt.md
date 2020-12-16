
# Hilt

* 原始文档： https://developer.android.com/training/dependency-injection/hilt-android

[TOC]


## 概述
* 是一个依赖注入库，减少手工管理依赖时需要的模板代码
* 提供另一个标准的方式来使用依赖注入
    * 为每个android类提供一个容器
    * 自动管理它们的生命周期
* 构建在Dagger库之上的


## 步骤

### 添加依赖
* 增加 hilt-android-gradle-plugin 插件
* 应用 hilt-android-gradle-plugin 插件
* 增加 库依赖
* 启用Java 8

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


### 创建application类
* 需要包含一个被@HiltAndroidApp标注的Application类
* @HiltAndroidApp会触发代码生成，包含一个基类作为应用层次的依赖容器
* 生成的Hilt组件会附加到Application对象的生命周期中，并提供了对它的依赖
* 该组件是应用程序的父组件，意味着任何其他组件都可以访问它提供的依赖项

```kotlin
@HiltAndroidApp
class ExampleApplication : Application() { ... }
```


### 注入依赖到android系统类
* 使用@AndroidEntryPoint标注来给其他类提供依赖项
* 如果一个系统类被标注了，依赖于该类的其他系统类也需要被标注
* @AndroidEntryPoint会给每个系统类生成一个独立的Hilt组件
* 使用@Inject标注来添加其他组件的依赖
* 目前支持几种系统类
    * Application
    * Activity
    * Fragment
    * View
    * Service
    * BroadcastReceiver

```kotlin
@AndroidEntryPoint
class ExampleActivity : AppCompatActivity() { 

    @Inject lateinit var analytics: AnalyticsAdapter

    ... 
}
```

### 定义绑定
* 为了实现依赖注入，Hilt需要知道如果如果从对应组件中提供依赖的实例
* 一个binding包含了足够的信息，来完成该工作
* 一种方法是使用Hilt的构造函数注入：使用@Inject标准类的构造函数
* 被标注的构造函数的参数会被认为是该类的依赖

```kotlin
class AnalyticsAdapter @Inject constructor(
  private val service: AnalyticsService
) { ... }
```

#### 注意
* 在构建阶段，Hilt会给android类生成Dagger组件
* 然后，Dagger会遍历应用代码，完成以下步骤
    * 生成、校验依赖关系图：保证这里没有未得到满足的依赖和依赖循环
    * 生成类：用于在运行阶段创建真实的对象和它们的依赖项



## Hilt模块

### 概述
* 部分情况下无法使用构造注入，这时候可以使用Hilt模块
    * 接口类
    * 第三库的，没有源代码的类
* 是一个使用了@Module的类，告诉Hilt如何生成特定类的实例
* 必须使用@InstallIn来告知Hilt该模块被用于那个系统类上
* Hilt模块提供的依赖对于系统类（被install上）生成所有组件都有效


### 用@Binds注入接口实例
* 使用@Binds来注入一个接口类的实例
* @Binds告知Hilt可以使用哪个实现类来给接口类提供实例
    * 函数返回类型告诉Hilt是哪个接口类
    * 函数参数告诉Hilt是哪个实现类


```kotlin
interface AnalyticsService {
  fun analyticsMethods()
}

class AnalyticsServiceImpl @Inject constructor(
  ...
) : AnalyticsService { ... }

@Module
@InstallIn(ActivityComponent::class)
abstract class AnalyticsModule {

  @Binds
  abstract fun bindAnalyticsService(
    analyticsServiceImpl: AnalyticsServiceImpl
  ): AnalyticsService
}
```


### 用@Provides注入实例
* 用于外部库，没有源代码的情况下注入依赖
* 通过@Provides标注的函数，来告诉Hilt如何提供实例
    * 函数返回类型告诉Hilt是哪个类型
    * 函数参数告诉Hilt对应的依赖项
    * 函数体告诉Hilt如何提供实例。每次需要提供实例时，Hilt都会执行一次该函数



```kotlin
@Module
@InstallIn(ActivityComponent::class)
object AnalyticsModule {

  @Provides
  fun provideAnalyticsService(
    // Potential dependencies of this type
  ): AnalyticsService {
      return Retrofit.Builder()
               .baseUrl("https://example.com")
               .build()
               .create(AnalyticsService::class.java)
  }
}
```

### 使用限定符实现多重绑定
* @Qualifier可以给同一类型绑定多个不同的实现
* @Qualifier是一个标注，用于标识出同一类型的多重绑定

#### 最佳实践
* 如果要给一个类型添加Qualifier，则需要给所有可能提供依赖的方法中添加Qualifier
* 如果留下一个基础、通过实现没有添加Qualifier，则非常容易导致出错，会让Hilt注入错误的依赖

```kotlin
//定义标注
@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class AuthInterceptorOkHttpClient

//定义provide
@Module
@InstallIn(ApplicationComponent::class)
object NetworkModule {

  @AuthInterceptorOkHttpClient
  @Provides
  fun provideAuthInterceptorOkHttpClient(
        authInterceptor: AuthInterceptor
  ): OkHttpClient {
      return OkHttpClient.Builder()
               .addInterceptor(authInterceptor)
               .build()
  }
}

//注入
class ExampleServiceImpl @Inject constructor(
  @AuthInterceptorOkHttpClient private val okHttpClient: OkHttpClient
) : ...
```


### 预定义限定符
* Hilt提供了一些预定义的Qualifier
* 对于Context类，Hilt提供了@ApplicationContext, @ActivityContext 两个限定符

```kotlin
class AnalyticsAdapter @Inject constructor(
    @ActivityContext private val context: Context,
    private val service: AnalyticsService
) { ... }
```


## 生成的组件

### 组件
* 对于每个需要进行属性注入的android类，都需要用@InstallIn来分配一个Hilt组件
* 每个Hilt组件用于注入绑定到对应的android类中



| Hilt组件 | 注入的类 |
| -- | -- | 
| ApplicationComponent | Application |
| ActivityRetainedComponent	| ViewModel |
| ActivityComponent	| Activity | 
| FragmentComponent	| Fragment | 
| ViewComponent	| View |
| ViewWithFragmentComponent	|  @WithFragmentBindings 标注的View |
| ServiceComponent	| Service | 


### 组件生命周期
* Hilt根据对应android类的生命周期，自动创建和销毁组件的实例

| Hilt组件 | 创建时机点 | 销毁时机点 |
| -- | -- | -- | 
| ApplicationComponent | Application#onCreate() | Application#onDestroy() |
| ActivityRetainedComponent	| Activity#onCreate() | Activity#onDestroy() |
| ActivityComponent	| Activity#onCreate() | Activity#onDestroy() | 
| FragmentComponent	| Fragment#onCreate() | Fragment#onDestroy() |
| ViewComponent	| View#super() | View destroyed |
| ViewWithFragmentComponent	|  View#super() | View destroyed |
| ServiceComponent	| Service#onCreate() | Service#onDestroy() |


### 组件作用域
* 默认Hilt组件都是没有作用域的。每次需要绑定时，Hilt都会创建一个实例
* Hilt允许一个对特定的组件带作用域的绑定
* Hilt只会创建一个带作用域的绑定，一旦组件的每个实例都绑定该作用域
    * 这个绑定的所有请求都会共享相同的实例

| android类 | Hilt组件 | 作用域 |
| -- | -- | -- | 
| Application | ApplicationComponent | @Singleton |
| View Model | ActivityRetainedComponent | @ActivityRetained | 
| Activity | ActivityComponent | @ActivityScoped | 
| Fragment | FragmentComponent | @FragmentScoped | 
| View | ViewComponent | @ViewScoped | 
| @WithFragmentBindings 标注的View | ViewWithFragmentComponent | @ViewScoped |
| Service | ServiceComponent | @ServiceScoped |

#### 例子1
* Hilt会只提供一个AnalyticsAdapter实例，在对应Activity的生命周期中


```kotlin
@ActivityScoped
class AnalyticsAdapter @Inject constructor(
  private val service: AnalyticsService
) { ... }
```

#### 例子2
* 无论何时当组件需要需要提供一个AnalyticsService实例时，Hilt只会提供一个相同的实例

```kotlin
@Module
@InstallIn(ApplicationComponent::class)
abstract class AnalyticsModule {
  @Singleton
  @Binds
  abstract fun bindAnalyticsService(
    analyticsServiceImpl: AnalyticsServiceImpl
  ): AnalyticsService
}

@Module
@InstallIn(ApplicationComponent::class)
object AnalyticsModule {
  @Singleton
  @Provides
  fun provideAnalyticsService(): AnalyticsService {
      return Retrofit.Builder()
               .baseUrl("https://example.com")
               .build()
               .create(AnalyticsService::class.java)
  }
}
```

### 组件层级
* 当安装一个module到一个组件中
* 在该组件内，允许该模块内的绑定作为其他绑定项的依赖进行访问
* 在任何该组件的子组件内也允许

![](https://gitee.com/cc12703/figurebed/raw/master/img/hilt-hierarchy.svg)


### 组件的默认绑定
* 每个Hilt组件都有一组默认绑定，Hilt会将它们和你自定义的绑定一起作为依赖进行注入
* 这些绑定都对应一个通用的activity, fragment类型，不是具体的子类型
* Hilt只使用一个activity组件定义来注入所有的activity，每个activity都拥有该组件的不同实例


| Hilt组件 | 默认绑定 |
| -- | -- | 
| ApplicationComponent | Application |
| ActivityRetainedComponent	| Application |
| ActivityComponent	| Application, Activity | 
| FragmentComponent	| Application, Activity, Fragment | 
| ViewComponent	| Application, Activity, View |
| ViewWithFragmentComponent	|  Application, Activity, Fragment, View | 
| ServiceComponent	| Application, Service |  


#### 例子
```kotlin
class AnalyticsServiceImpl @Inject constructor(
  @ApplicationContext context: Context
) : AnalyticsService { ... }

//没有用限定符
class AnalyticsServiceImpl @Inject constructor(
  application: Application
) : AnalyticsService { ... }
```

```kotlin
class AnalyticsAdapter @Inject constructor(
  @ActivityContext context: Context
) { ... }

//没有用限定符
class AnalyticsAdapter @Inject constructor(
  activity: FragmentActivity
) { ... }
```


## Hilt未支持类

* Hilt本身支持了大部分的android类，但是仍然需要处理一些Hilt不支持的类
* 这个时候你需要使用@EntryPoint枚举

### @EntryPoint
* 这个入口点定义了一个边界，在Hilt管理的代码和不管理的代码之间。
* 从这个点开始，代码将进入Hilt管理的对象依赖图中
* 这个入口点允许Hilt使用外部代码来在依赖图中提供依赖

### 例子
* 定义一个接口类，使用@EntryPoint进行标注
* 使用EntryPointAccessors中的静态方法来访问入口点

```kotlin
//定义
class ExampleContentProvider : ContentProvider() {
  @EntryPoint
  @InstallIn(ApplicationComponent::class)
  interface ExampleContentProviderEntryPoint {
    fun analyticsService(): AnalyticsService
  }
  ...
}

//使用
class ExampleContentProvider: ContentProvider() {
    ...
  override fun query(...): Cursor {
    val appContext = context?.applicationContext ?: throw IllegalStateException()
    val hiltEntryPoint = EntryPointAccessors.fromApplication(appContext, 
                  ExampleContentProviderEntryPoint::class.java)
    val analyticsService = hiltEntryPoint.analyticsService()
    ...
  }
}

```

## Hilt和Dagger

### 概述
* Hilt是建立在Dagger这个依赖注入库之上的
* 提供了一个标准的方法将Dagger整合进android应用中
* Hilt可以减少Dagger相关的模块代码的书写

### Hilt目标
* 简化Dagger相关的基础设施
* 创建一个组件和作用域的标准集，用于简化设置、提高可读性、在应用间共享代码
* 提供一个简单的方法在不同的绑定之间进行切换（测试环境、调试环境、发布环境）

### Hilt自动生成和预置
* 集成android框架类的组件：Dagger中需要手动创建
* 作用域标注
* 预定义绑定
* 预定义作用域：@ApplicationContext, @ActivityContext