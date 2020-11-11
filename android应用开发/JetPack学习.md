

# JetPack学习

## 应用架构

### 原则

#### 关注点分离
* 不要再activity，fragment中编写所有代码
* 这些类中应仅包含处理界面和操作系统交互的逻辑

#### 通过模型驱动界面
* 模式用于处理应用数据，独立于应用的界面
* 理想选择，是使用持久性模型


### 推荐架构

#### 架构图
![](https://gitee.com/cc12703/figurebed/raw/master/img/20201111113003.png)


#### 界面

##### 组成
* xxx.xml 界面布局
* xxxFragment 界面控制器
* xxxViewModel 提供数据

##### 说明
* ViewModel 为特定的界面组件提供数据，并包含业务逻辑，与模型进行通信
* LiveData 是一种可观察的数据存储器，用于将ViewModel连接到Fragment上


#### 获取数据
* ViewModel会将数据获取过程委派给一个新的模块，即存储区
* 存储区（repository）会处理数据操作，提供一个干净的API，以便应用的其他部分可以获取数据


##### 管理组件依赖
* 依赖注入：能够定义依赖项而不构造它们，在运行时由其他类来提供这些依赖项
* 服务定位器：提供了一个注册表，类可以从中获取依赖项，而不构造它们

##### 单一可信来源
* 将存储区的某个数据源指定为应用其余部分的单一可信来源


#### 显示正在执行的操作
* 返回一个LiveData类型的对象，该对象包含网络操作的状态（NetworkBoundResource）
* 在repository中提供一个可以返回数据刷新状态的公共函数


#### 测试
* 界面和交互：使用 界面插桩测试，（Espresso库）
* ViewModel：使用 JUnit 进行测试
* Repository：使用 JUnit 进行测试


### 最佳实践

* 避免将应用的入口点（Activity,Service,receiver）指定为数据源
* 在各个模块之间设置明确的职责界限
* 尽量少公开每个模块的代码
* 考虑如何使每个模块可以独立测试
* 保留尽可能多的相关数据和最新数据
* 将一个数据源指定为单一可信来源


## 组件

### 数据绑定
* 以声明方式将可观察数据绑定到界面元素


### Lifecycles

#### 功能
* 管理Activity,Fragment的生命周期
* Lifecycle类，用于储存相关组件的生命周期状态信息


#### 生命周期
![](https://gitee.com/cc12703/figurebed/raw/master/img/20201111113608.png)


#### 监听者

* 需要实现LifecycleObserver接口

```kotlin
class MyObserver : LifecycleObserver {

    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    fun connectListener() { .. }

}

myLifecycleOwner.getLifecycle().addObserver(MyObserver())
```

#### 提供者

* 需要实现LifecycleOwner接口
* androidx中的Fragment和activity已经实现该接口
* 自定义实现时，可以使用LifecycleRegistry类


```kotlin
class MyActivity : Activity(), LifecycleOwner {

    private lateinit var registry
    
    override fun onCreate(state: Bundle?) {
        super.onCreate(stete)
        
        registry = LifecycleRegistry(this)
        registry.markState(Lifecycle.State.CREATED)
    }
    
    override fun onStart() {
        super.onStart()
        
        registry.markState(Lifecycle.State.STARTED)
    }
    
    override fun getLifecycle(): Lifecycle {
        return registry
    }
}
```

#### 最佳实践
* 使界面控制器（activity, fragment）尽可能保持精简
* 设法编写数据驱动型界面
* 将数据逻辑放在viewModel中，但是其不负责获取数据
* viewModel应调用其他组件来获取数据，然后将结果提供给界面控制器
* 使用data binding在视图与界面控制器之间保持干净的接口
* 如果界面很复杂，可以创建presenter来处理界面修改
* 避免在viewModel中引用View或Activity上下文



### LiveData

#### 功能
* 在数据更改时通知视图

#### 优势
* 确保界面符合数据状态
* 不会发生内存泄漏
* 不会因activity停止而导致崩溃
* 不在需要手动处理生命周期
* 数据始终保持最新状态
* 共享资源

#### 使用
* 创建LiveData实例，通常在ViewModel中完成
* 创建Observer对象，实现onChanged()方法
* 使用observe()将Observer对象加入LiveData对象

#### 创建
* LiveData是一种可以用于任何数据的封装容器

```kotlin
class NameViewModel : ViewModel() {
    val currentName: MutableLiveData<String> by lazy {
        MutableLiveData<String>()
    }
}
```

#### 观察
* 大多数情况下，可以在onCreate()中配置观察者
* LiveData仅在数据发生更改时才发送更新，并仅发送给活跃观察者

```kotlin
class NameActivity : AppCompatActivity() {

    override fun onCreate(state: Bundle?) {
        super.onCreate(state)
        
        val nameObserver = Observer<String> { newName ->
            nameTextView.text = newName
        }
        
        model.currentName.observe(this, nameObserver)
    }

}
```

#### 更新
* MutableLiveData使用setValue(), postValue()来更新数据
* setValue()只能在主线程中调用
* postValue()可以在其他线程中调用

#### 扩展
* 当LiveData有活跃观察者时，会调用onActive()方法
* 当LiveData无任何活跃观察者时，会调用onInactive()方法
* setValue()将更新值，并通知给任何活跃观察者

#### 转换
* 使用Transformations.map() 对存储的值应用函数，并将结果传播到下游
* 使用Transformations.switchMap() 对存储的值应用函数，并将结果解封和分派到下游


#### 合并多个源
* 使用MediatorLiveData类
* 只要任何原始的源对象发生更改，就会触发该对象的观察者


### 其他

* Navigation 应用内导航
* Paging 按需加载数据
* Room 访问sqlite数据库
* ViewModel 以注重生命周期的方式管理界面相关的数据
* WorkManager 管理后台作业