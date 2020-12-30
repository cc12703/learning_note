


# LiveData对象Data

* 原始文档： https://developer.android.com/topic/libraries/architecture/livedata

[TOC]

## 总体

### 概述
* LiveData是一个可观察的数据持有类
* 不像正规的可观察者，LiveData是生命周期敏感的
* 响应其他应用组件的生命周期（activity, fragment, service）
* 这种敏感性保证了LiveData只会在组件激活的时候，通知该组件

### 观察者
* LiveData认为一个观察者，是实现了Observer类， 并在生命周期是STARTED, RESUMED时，才会处于激活状态
* LiveData只会通知激活状态下的观察者，那些未激活观察者是不会收到变更通知的

### 自动移除观察
* 你可以注册一个观察者，与实现LifecycleOwner接口的对象配对
* 配对关系允许观察者可以被移除，当对应的Lifecycle对象变更为DESTROYED时
* 这个特性对于activity, fragment非常有用，它们可以安全的观察LiveData对象而不用担心资源泄漏问题


## 优势

### 保证界面与数据匹配
* LiveData遵循观察者模式
* 当生命周期变更时，LiveData会通知观察者
* 在这些观察者对象中，你可以写代码去更新界面。
* 观察者可以在他们变更时去更新界面，而不用每次应用数据变更时都去更新界面

### 无内存泄漏
* 观察者绑定了Lifecycle对象
* 当相关的lifecycle被销毁时，观察者可以清除他们自己

### 不会因为activity停止而崩溃
* 如果观察者的生命周期是未激活，例如：activity在后台堆栈中
* 这些观察者将不会接收到LiveData的事件

### 不需要手动管理生命周期
* 界面组件只需要观察相关的数据，而不需要处理停止、恢复观察
* LiveData会自动管理这些，因为它是生命周期敏感的

### 一直都是最新数据
* 如果生命周期变成未激活，当它重新变成激活时会接收到最新的数据

### 合适的配置变更
* 如果activity、fragment因为配置变更而被重建了（像屏幕旋转）
* 它立刻就会接收到最新的有效数据

### 共享资源
* 可以使用单例模式来扩展一个LiveData对象，用来包装系统服务。 这样就可以在应用中共享这些系统服务
* 一旦一个LiveData对象与系统服务相链接，任何需要该资源的观察者都可以去观测这个LiveData对象


## 使用方法

### 使用步骤
1. 创建一个LiveData对象，保存特定的数据。这个一般在ViewModel中进行
1. 创建一个观察者对象，并实现onChanged()方法
    * 该方法用于响应当LiveData保存数据变更时，该如何操作
    * 创建一般在界面控制器中进行，像：activity, fragment
1. 使用observe()来使观察者对象连接到一个LiveData对象
    * 这个方法接受一个LifecycleOwner对象作为参数
    * 连接动作一般在界面控制器中进行

### 说明
* 当前你更新LiveData中的数据时，LiveData会触发所有已注册的观察者，只要连接的LifecycleOwner在激活状态
* LiveData允许界面控制器观察者订阅更新。当LiveData中的数据发生变更，界面就会自动更新


### 创建LiveData对象
* LiveData是一个包装类，可以用于任何数据。包括实现了Collections接口的对象
* 一个LiveData对象一般保存在ViewModel对象中

#### 例子
```kotlin
class NameViewModel : ViewModel() {

    // 创建一个字符串的LiveData
    val currentName: MutableLiveData<String> by lazy {
        MutableLiveData<String>()
    }

}
```

#### 在ViewModel中的优点
* 避免activity、fragment太臃肿
* 解耦LiveData对象和具体的activity，fragment对象绑定


### 观察LiveData对象
* 大部分情况下，会在应用组件的onCreate()中开始观察一个LiveData对象
    * 原因1：确保系统不会重复调用
    * 原因2：确保activity、fragment在变成激活时马上就数据可以显示
* 大部分情况下，LiveData只会在数据变更时分发更新消息给激活状态的观察者
    * 例外：观察者从未激活状态变成激活状态，会接收到一个更新消息，不管数据是否有变更


#### 例子
```kotlin
class NameActivity : AppCompatActivity() {

    private val model: NameViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // 创建一个观察者用于更新界面
        val nameObserver = Observer<String> { newName ->
            nameTextView.text = newName
        }

        // 观察一个LiveData对象
        model.currentName.observe(this, nameObserver)
    }
}
```
说明
* 在observe()被调用后，onChanged()会被马上触发，以提供保存在currentName中的最新值
* 如果LiveData对象没有在currentName中设置值，则onChanged()不会被调用


### 更新LiveData对象
* LiveData没有提供公开方法来更新保存的数据
* MutableLiveData提供了公开方法setValue(T)和postValue(T)，用于更新数据
* 通常情况下，MutableLiveData也在ViewModel中使用，ViewModel只导出不可变的LiveData给观察者

```kotlin
button.setOnClickListener {
    val anotherName = "John Doe"
    model.currentName.setValue(anotherName)
}
```
注意
* 在主线程中，必须调用setValue(T)来更新LiveData对象
* 如果代码运行在工作线程上，你需要使用postValue(T)来更新LiveData对象

### 使用Room
* Room持久化库支持可观察性的查询，会返回LiveData对象。是DAO中的一部分
* Room会生成必要的代码，当数据库被更新时自动更新LiveData
* Room会在后台线程运行异步查询。这种模型对于数据需要在界面上显示并和数据库中数据保持同步非常有效




## 扩展
* LiveData会认为观察者是激活的，当观察者的生命周期处于STARTED，RESUMED状态时

### 重要方法
#### onActive()
* 当LiveData有一个激活的观察者时会被调用
* 你需要在方法中启动监测

#### onInactive()
* 当LiveData中没有任何一个激活的观察者时会被调用
* 你需要在方法中关闭监测

#### setValue()
* 更新LiveData中的数据
* 通知激活的观察者数据已变更


### 例子
```kotlin
//创建
class StockLiveData(symbol: String) : LiveData<BigDecimal>() {
    private val stockManager = StockManager(symbol)

    private val listener = { price: BigDecimal ->
        value = price
    }

    override fun onActive() {
        stockManager.requestPriceUpdates(listener)
    }

    override fun onInactive() {
        stockManager.removeUpdates(listener)
    }

    //支持单例模式
    companion object {
        private lateinit var sInstance: StockLiveData

        @MainThread
        fun get(symbol: String): StockLiveData {
            sInstance = if (::sInstance.isInitialized) sInstance else StockLiveData(symbol)
            return sInstance
        }
    }
}

//使用
public class MyFragment : Fragment() {
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        val myPriceListener: LiveData<BigDecimal> = ...
        myPriceListener.observe(viewLifecycleOwner, Observer<BigDecimal> { price: BigDecimal? ->
            // Update the UI.
        })

        StockLiveData.get(symbol).observe(viewLifecycleOwner, Observer<BigDecimal> { price: BigDecimal? ->
            // Update the UI.
        })
    }
}
```
说明
* observe()会传入一个LifecycleOwner对象作为第一个参数，该对象已关联到fragment的视图
* 这个操作意味着这个观察者已经绑定到了Lifecycle对象
    * 如果Lifecycle对象不在激活状态，数据变更了该观察者也不会被回调到
    * 在Lifecycle对象被销毁后，这个观察者会自动被移除
* LiveData的生命周期感知特性，可以让LiveData在多个activity,fragment,service之间共享




## 转换器

### 需求
* 在分发给观察者之前，需要修改LiveData中保存的值
* 需要基于另一个LiveData中的值，返回一个不同的LiveData对象


### 方法
#### Transformations.map()
* 将方法应用于LiveData中的值
* 并向下游传导该值

```kotlin
val userLiveData: LiveData<User> = UserLiveData()
val userName: LiveData<String> = Transformations.map(userLiveData) {
    user -> "${user.name} ${user.lastName}"
}
```

#### Transformations.switchMap()
* 类似于map(),将方法应用于LiveData中的值
* 解开对象并向下游分发结果
* 传给switchMap的方法必须返回一个LiveData对象

```kotlin
private fun getUser(id: String): LiveData<User> {
  ...
}
val userId: LiveData<String> = ...
val user = Transformations.switchMap(userId) { id -> getUser(id) }
```

### 特点
* 转换器是延迟计算的，直到有观察者观察了返回的LiveData对象，才会进行计算
* 可以使用转换器方法，跨越观察者生命周期携带数据

### ViewModel中使用
* 如果在ViewModel中需要一个Lifecycle对象，一个转换器可能是最好的方案


```kotlin
class MyViewModel(private val repository: PostalCodeRepository) : ViewModel() {
    private val addressInput = MutableLiveData<String>()
    val postalCode: LiveData<String> = Transformations.switchMap(addressInput) {
            address -> repository.getPostCode(address) 
    }


    private fun setInput(address: String) {
        addressInput.value = address
    }
}
```
说明
* postalCode属性是定义成一个addressInput的转换器
* 只要有一个激活的观察者关联到postalCode属性上，该属性的值就会被重新计算，无论何时addressInput变更了，值会被重新获取
* 这个机制允许应用的低层次模块可以创建LiveData对象，并在需要时懒计算
* ViewModel对象可以很方便的获取LiveData对象引用，并创建一个转换器


### 创建新的转换器
* 在应用中可能需要使用一打的不同种类的转换器，但是默认实现不会提供
* 要实现自定义转换器，可以使用MediatorLiveData类
    * 该类监听其他LiveData对象，并处理发送给它们的事件
    * 会正确的传递源LiveData的状态



## 合并多个源
* MediatorLiveData是LiveData的子类，允许合并多个LiveData源
* 只要其中任何一个源LiveData变更了，MediatorLiveData的观察者都会被触发

### 例子
* 有一个LiveData可以被本地数据库和网络进行更新
* 可以向MediatorLiveData增加以下两个源
    * 一个LiveData关联数据库中的数据
    * 一个LiveData关联网络中获取的数据
* activity只需要观察MediatorLiveData对象，就可以接收来自两个源的更新