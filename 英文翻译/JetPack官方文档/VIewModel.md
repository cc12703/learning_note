
# ViewModel

* 参考资料： https://developer.android.com/topic/libraries/architecture/viewmodel

[TOC]

## 总体

### 概述
* ViewModel被设计用来存储、管理界面相关的数据，以生命周期感知的方式
* ViewModel允许数据在配置改变后而存活下来

### 界面控制器
* android框架管理这界面控制器的生命周期，像：activity, fragment
* 框架会因为特定用户操作、设备事件而销毁、重建界面控制器。会完全脱离你的控制

#### 问题-丢失数据
* 如果系统销毁、重建一个界面控制器，任何保存在其中的数据都会丢失
    * 例子：如果应用在activity中保存了用户列表，当该activity因为配置变更而重建时，需要重新获取一下用户列表
* 对于简单数据，可以使用onSaveInstanceState()来保存，并在onCreate()时进行恢复。该方法只适用于小量的可以序列化的数据

#### 问题-频繁的异步调用
* 界面控制器需要频繁地进行异步调用，需要等待一段时间才能返回
* 界面控制器需要管理这些调用，确保系统在销毁它们后可以清除它们，避免潜在的内存泄漏
* 这些管理需要大量的维护工作，在这个案例中，对象因为配置变更而被重建，不得不重新调用接口已获取资源

#### 问题-承担太多功能
* 界面控制器的首要作用是显示数据、响应用户行为、处理操作系统交互
* 如果要需要从数据库、网络加载数据会导致类的膨胀
* 如果给界面控制器赋予太多责任的话，会导致单个类处理了大多一个应用的功能

## 实现
* 架构组件提供了ViewModel类，是界面控制器的助手类，用于负责为界面准备数据
* ViewModel对象会在配置变更时自动保存，所在在activity, fragment重新创建后会立即有效

### 例子
```kotlin
//创建
class MyViewModel : ViewModel() {
    private val users: MutableLiveData<List<User>> by lazy {
        MutableLiveData().also {
            loadUsers()
        }
    }

    fun getUsers(): LiveData<List<User>> {
        return users
    }

    private fun loadUsers() {
        // 进行一次异步操作，获取用户列表
    }
}

//使用
class MyActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        // 在onCreate()中创建ViewModel，在activity重建后会接收到相同的MyViewModel实例

        val model: MyViewModel by viewModels()
        model.getUsers().observe(this, Observer<List<User>>{ users ->
            // 更新界面
        })
    }
}
```

### 特点
* ViewModel被设计成脱离view和LifecycleOwner独立存在
* 这个设计意味着可以更容易的为其写测试用例，而不需要任何view和Lifecycle对象
* ViewModel可以包含LifecycleObservers，像LiveData对象
* ViewModel不能观察任何生命周期感知的可观察者
* 如果ViewModel需要一个Application上下文，则可以继承AndroidViewModel类（构建函数中可以接收Application对象）


## 生命周期
### 概述
* ViewModel对象的作用域取决于获取该对象时传给ViewModelProvider的Lifecycle
* ViewModel会一直存在在内存中，直到其限定的Lifecycle永久的消失了
    * 对于activity，就是其被finish
    * 对于fragment，就是其被detach

### 图示

说明
* 展示了activity从旋转屏幕到被结束时的生命周期状态
* ViewModel会在首次请求时就被创建，然后一直存在直到activity被销毁、结束

![](https://gitee.com/cc12703/figurebed/raw/master/img/20210104113248.png)


## 在fragment间共享数据
### 案例
* 一个普遍情况是：activity中存在多个fragment，并且之间需要进行通信
* 例子：分离界面(主界面-详情界面)的多个fragment
    * 一个fragment显示列表信息，用户可以从中选择一个列表项
    * 另一个fragment显示列表项的内容

#### 以前方案
* 所有的fragment需要定义一些接口
* activity必须要将多个fragment绑定起来
* fragment需要处理其他fragment未创建、不可见的情况

#### 新方案示例
```kotlin
class SharedViewModel : ViewModel() {
    val selected = MutableLiveData<Item>()

    fun select(item: Item) {
        selected.value = item
    }
}

class MasterFragment : Fragment() {
    private lateinit var itemSelector: Selector
    private val model: SharedViewModel by activityViewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        itemSelector.setOnClickListener { item ->
            //更新界面
        }
    }
}

class DetailFragment : Fragment() {
    private val model: SharedViewModel by activityViewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        model.selected.observe(viewLifecycleOwner, Observer<Item> { item ->
            // 更新界面
        })
    }
}
```
说明
* 多个fragment可以共享ViewModel来进行通信

#### 优点
* activity不需要做任何事，不需要知道通信的细节
* 多个fragment不需要知道彼此的存在
    * 如果一个fragment消失了，其他的fragment还可以继续正常运行
* 每个fragment都有自己的生命周期，彼此之间不会相互影响


## 替换Loader
### 概述
* Loader像CursorLoader会被频繁用来在应用界面和数据库之间保持数据的同步
* 你可以使用ViewModel和一些其他类，来替换掉Loader
* 使用ViewModel可以将界面控制器从数据加载操作中分离出来，意味着类之间有着更少的引用


### 以前的方案
* CursorLoader用于观察数据库的内容
* 当数据库值变更时，loader会自动触发，并重载数据来更新界面

图示
![](https://gitee.com/cc12703/figurebed/raw/master/img/20210105102051.png)

### 新方案
* ViewModel结合Room和LiveData可以替换掉loader
* ViewModel保证数据在设备配置变更时可以存活下来
* 当数据库值变更时，Room会通知LiveData
* LiveData会使用变更后的值更新界面

图示
![](https://gitee.com/cc12703/figurebed/raw/master/img/20210105102246.png)