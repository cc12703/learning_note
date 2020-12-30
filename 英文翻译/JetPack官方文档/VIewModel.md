
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

![](https://gitee.com/cc12703/figurebed/raw/master/img/20210104113248.png)

