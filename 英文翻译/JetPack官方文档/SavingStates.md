

# SavingStates

* 原始文档：https://developer.android.com/topic/libraries/architecture/saving-states



## 保存UI状态

### 概述
* 及时的保存和恢复一个activity的UI状态，跨越系统启动的activity和application的销毁，是用户体验中至关重要的一部分
* 在这些情况下，用户希望UI状态可以恢复到和原来一样，但是系统会破坏activity和任何存储其中的状态
* 为了跨越用户体验和系统行为之间的鸿沟，需要整合ViewModel对象，onSaveInstanceState()和本地存储，来保存跨应用和activity的UI状态
* 如何整合这些，依赖于UI数据的复杂度、用户使用情况、恢复速度和内存使用的比值
* 无论选择什么方案，都需要确保应用满足用户的期望，关心他们UI的状态，提供一个顺滑、高效的UI


### 用户体验和系统行为
* 根据用户采取的行为，他们希望有时候清除activity的状态，有时候保存activity的状态
* 在某些情况下，系统的自动行为符合用户的期望。但在另一些情况下，系统行为和用户的期望相反

#### 用户触发UI状态的丢弃
* 用户希望他们打开的activity，其瞬间的UI状态可以保持住直到他们完全丢弃这个activity
* 用户假定在完全丢弃的时候，是永久性的离开这个activity。当他们重新打开这个activity时，是一个完全干净的状态
* 这时候底层系统的行为刚好符合用户的期望：activity实例会被销毁，从内存中移除，包含任何存储在其中的状态、保存的状态记录

##### 丢弃触发方式
* 按回退按键
* 将activity交换出屏幕
* 从activity导航出去
* 在设置界面杀死应用


#### 系统触发UI状态的丢弃
* 用户会希望在配置变更后，activity状态能恢复到和以前一样
    * 像屏幕旋转、切换到多窗口模式
* 系统在这种情况下的做法是销毁activty，并清除相关的任务UI状态
* 用户也会希望在他们临时切换到别的应用后在回来时，activity的状态能恢复到和以前一样
    * 像用户在搜索界面进行搜索，按home键或者接电话，当他在次回到搜索界面时希望搜索的关键词和结果都在
* 在这种场景下，你的应用会被放置在后台，系统会尽可能的将它们保存在内存中
    * 但是当用户进入其他应用时，系统有可能会销毁你应用的进程
    * 这时候activity实例会被销毁，包含其中的任何状态
    * 当用户再次启动你的应用时，activity会进入一个出乎意料的干净状态


### 保存UI状态的选项
* 当用户关于UI状态的期望和系统行为不符合时，你就必须保存和恢复UI的状态

| 维度  | ViewModel | SavedInstanceState | 持久化储存 |
| -- | -- | -- | -- |
| 存储地方| 内存 | 串行化到磁盘 | 磁盘、网络 |
| 配置变更是否存活 | 是 | 是 | 是 | 
| 应用进程死亡是否存活 | 否 | 是 | 是 | 
| 完全activity退出是否存活 | 否  | 否 | 是 |
| 数据限制 | 可以保存复杂对象 | 只能保存原始类型和简单对象 | 可以保存对象 |
| 读/写时间 | 非常快 | 慢 | 慢 |


### 使用ViewModel处理配置变更
* 当用户在主动使用应用时，ViewModel是存储和管理UI相关数据的理想方案
    * 允许你在发生配置变更时（屏幕旋转、窗口缩放），快速的读取UI状态，而不用通过磁盘和网络
* ViewModel将数据保存在内存中，意味着恢复比磁盘和网络更便宜
* ViewModel会关联到一个activity上，在配置变更时会保存在内存中，系统会自动重新关联一个新的activity实例
* ViewModel会被系统自动销毁，当用户从activty退出、你调用了finish()
    * 保存的状态会被清除
* 不像 SavedInstanceState，ViewModel会在系统杀死应用进程的过程中被销毁
    * 需要和onSaveInstanceState()一起使用
    * 在SavedInstanceState中存储标识符，以便在进程死亡后帮助ViewModel重载数据


### 使用onSaveInstanceState()处理系统杀死应用进程
* onSaveInstanceState()回调会保存数据，用于重载UI控制器状态(像activity,fragment)
* SavedInstanceState的存储空间和速度都会受到限制，因为onSaveInstanceState()需要串行数据到磁盘
    * 串行化会消耗大量内存，如果对象比较复杂的话
    * 整个过程在主线程中进行，长时间的串行化会导致丢帧
* 不要用使用onSaveInstanceState()来存储大量的数据，像：位图，不太复杂的数据结构
    * 这些都需要长时间的串行化和反串行化
* 可以使用onSaveInstanceState()来存储原始数据类型和简单的、小的对象，像：字符串
* 可以使用onSaveInstanceState()来存储一下必要的小量的数据，像：ID号
    * 为了重建必要数据，以便在其他持久化机制失败时，将UI恢复到以前的状态
* 根据应用的使用场景，你也可以完全不用onSaveInstanceState()
    * 像浏览器应用在退出应用前，会把用户引导到用户自己搜索的网页上
    * 如果你应用的行为类似浏览器，你可以不用onSaveInstanceState(),而用本地持久化

#### Intent触发
* 当你从一个Intent中打开activity时，当配置变更，系统恢复activity时，额外的数据包会发送给activity
* 如果一个小的UI状态数据，像搜索查询可以作为一个intent的额外数据传给activity


#### SavedStateRegistry
##### 概述
* 从fragment 1.1.0 或 activity 1.0.0开始，UI控制器就是实现了SavedStateRegistryOwner接口，并提供了一个和UI控制器绑定的SavedStateRegistry对象
* SavedStateRegistry允许组件挂钩进入UI控制器的**已保存状态**，读取并写入数据
    * 例子ViewModel的已保存状态模块，使用SavedStateRegistry创建了一个SavedStateHandle，提供给ViewModel对象使用
* 使用getSavedStateRegistry()来获取SavedStateRegistry


##### 保存数据
* 组件要给SavedState共享数据，需要实现SavedStateRegistry.SavedStateProvider
* SavedStateProvider定义了saveState()方法，该方法允许组件返回一个bundle，包含任意需要保存的状态
* SavedStateRegistry会在UI控制器的保存状态阶段调用该函数

```kotlin
class SearchManager : SavedStateRegistry.SavedStateProvider {
    companion object {
        private const val QUERY = "query"
    }

    private val query: String? = null

    ...

    override fun saveState(): Bundle {
        return bundleOf(QUERY to query)
    }
}
```

##### 注册供应者
* 调用SavedStateRegistry.registerSavedStateProvider()来注册一个供应者
    * 要传入一个和供应者数据关联的键
* 调用SavedStateRegistry. consumeRestoredStateForKey()可以获取指定供应者保存的数据
    * 要传入一个和供应者数据关联的键
* 在activity和fragment中
    * 一种方法，onCreate()中，在调用super.onCreate()后注册供应者
    * 另一种方法，给SavedStateRegistryOwner设置一个LifecycleObserver，在ON_CREATE事件中进行注册


```kotlin
class SearchManager(registryOwner: SavedStateRegistryOwner) 
            : SavedStateRegistry.SavedStateProvider {
    companion object {
        private const val PROVIDER = "search_manager"
        private const val QUERY = "query"
    }

    private val query: String? = null

    init {
        // 注册一个LifecycleObserver，处理ON_CREATE事件
        registryOwner.lifecycle.addObserver(LifecycleEventObserver { _, event ->
            if (event == Lifecycle.Event.ON_CREATE) {
                val registry = registryOwner.savedStateRegistry

                // 注册一个供应者，将来会调用saveState()
                registry.registerSavedStateProvider(PROVIDER, this)

                // 获取以前保存的数据
                val state = registry.consumeRestoredStateForKey(PROVIDER)
                query = state?.getString(QUERY)
            }
        }
    }

    override fun saveState(): Bundle {
        return bundleOf(QUERY to query)
    }
    ...
}

class SearchFragment : Fragment() {
    private var searchManager = SearchManager(this)
    ...
}
```


### 使用本地持久化处理进程死亡
* 持久化本地存储：像数据库、共享preferences，当应用被安装在设备上后会一直存在
    * 系统引起的activity死亡
    * 系统引起的应用进程死亡
* 读取数据比较费时，需要从本地存储中载入内存
* 大部分情况下，本地存储已经是你应用架构中的一部分了，存储不能丢失的数据
* 你应该使用ViewModel来临时保存暂时的UI状态，使用本地存储来保存其他应用数据


### 管理UI状态:分离和整合
* 要有效的保存和恢复UI状态，就需要把它们分离带不同类型的存储机制中
* 大部分情况下，每个机制都储存不同类型的数据，基础数据复杂度、存取速度、存活时间的权衡

#### 本地持久化
* 保存所有你不希望丢失的数据：歌曲对象的集合，包括音频文件和元数据

#### ViewModel
* 在内存中保存那些关联到UI控制器中用于显示的数据：最近搜索的歌曲对象、最近搜索查询信息

#### onSaveInstanceState()
* 存储小量的、容易重加载的ativity状态数据，用于系统重建UI控制器：最近的搜索查询信息
* 将复杂对象保存在本地存储中，将唯一ID保存在onSaveInstanceState


#### 例子
* 考虑一个activity，用于搜索你的歌曲库

##### 添加歌曲
* 当用户添加一首歌曲时，ViewModel会立刻委派在本地持久化这些数据
* 如果新加入的歌曲要在界面上显示，你需要在ViewMdoel中更新数据已反映歌曲的增加
* 记住，数据库的插入需要在主线程外进行

#### 搜索歌曲
* 当用户搜索一首歌曲时，无论多复杂的歌曲数据都要从数据库中加载，并立刻存入ViewModel中
* 你也需要将搜索查询本身保存进ViewModel中

#### 进入后台
* 当activity进入后台时，系统会调用onSaveInstanceState()
* 你将在onSaveInstanceState()的bundle中保存搜索查询本身
    * 小量数据更容易保存


### 恢复复杂状态：组装碎片
当用户重新回到上面的activity时，对于重建有两种可能的场景

#### 场景1
* 在被系统停止后，activity被重建了
* activity在onSaveInstanceState()的bundle中保存了搜索查询，将其传入ViewModel
* ViewModel发现其没有搜索结果，就委派外部加载对应的搜索结果

#### 场景2
* 在配置变更后，activity被新建了
* activity在onSaveInstanceState()的bundle中保存了搜索查询，将其传入ViewModel
* ViewModel依然缓存了对应的搜索结果，就不需要再次从数据库查询了



## ViewModel的状态保存

### 概述
* ViewModel已经可以处理配置变更，所以对应屏幕旋转之类的不需要再担心了
* 如果要处理系统杀死应用进程，你就需要使用onSaveInstanceState()来作为备份

#### 使用模板
* UI状态通常在ViewModel中保存和引用，而不是在activity中，所以需要一些使用模板
* 当使用本模块时，ViewModel需要在构造函数中接收一个SaveStateHandle对象
* 该对象是一个键值映射，用于从状态保存中读、写对象
* 这些值会在进程被杀后，由系统进行持久化保存


### 依赖
* 从fragment 1.2.0 或 activity 1.1.0开始，就可以在ViewModel的构造函数中使用SavedStateHandle了
* 创建ViewModel实例是不需要额外配置，默认的构造工厂已经提供了SavedStateHandle给ViewModel
* 当使用自定义ViewModelProvider.Factory实例时，可以通过集成AbstractSavedStateViewModelFactory来使用SavedStateHandle

```kotlin
class SavedStateViewModel(private val state: SavedStateHandle) 
            : ViewModel() { 
                ... 
}

class MainFragment : Fragment() {
    val vm: SavedStateViewModel by viewModels()

    ...
}
```

### SavedStateHandle的使用
* 该类是一个键值对映射，可以通过set(),get()方法来读、写数据
* 可以使用getLiveData()，将读取的数据包装在LiveData中
* 大部分情况下，值会在用户交互中被设置，像：输入查询字符串用来过滤列表数据


```kotlin
class SavedStateViewModel(private val savedStateHandle: SavedStateHandle) : ViewModel() {
    val filteredData: LiveData<List<String>> =
        savedStateHandle.getLiveData<String>("query").switchMap { query ->
        repository.getFilteredData(query)
    }

    fun setQuery(query: String) {
        savedStateHandle["query"] = query
    }
}
```

#### 其他方法
* contains(String key) ：检查是否包含该键
* remove(String key)：通过指定键移除数据
* keys()：获取所有的键


### 支持的数据类型
* SavedStateHandle中的数据会作为一个Bundle来处理


### 直接支持的类型
* 通过set(), get()来操作以下类型

| 类型 | 数组类型|
| -- | -- |
| double | double[] | 
| int | int[] | 
| long | long[] | 
| String | String[] | 
| byte | byte[] | 
| char | char[] | 
| float | float[] | 
| Parcelable | Parcelable[] | 
| Serializable | Serializable[] |
| short | short[] | 
| SparseArray || 
| Binder | |
| Bundle | |
| ArrayList | |

### 保存无法串行化的类
* 如果一个类型无法实现Parcelable或Serializeable类，就无法直接保存到SavedStateHandle中
* 从 lifecycle 2.3.0-alpha03开始，通过提供自定义的保存、恢复逻辑将对象转换成Bundle，就可以向SavedStateHandle中保存任何对象
* 使用 setSavedStateProvider() 来提供转换逻辑

#### SavedStateProvider
* 该类是一个接口类，只定义了一个saveState()方法
* 该方法返回一个Bundle对象，包含了你要保存的数据
* 当SavedStateHandle要保存状态时，会调用saveState()来获得该Bundle对象，用关联的键保存该Bundle

#### 例子
* 应用从摄像头应用中请求一张图片，传入一个临时文件用于保存图片
* 为了确保进程被杀死后临时文件不丢失，TempFileViewModel使用了SavedStateHandle来持久化数据
* 实现了一个SavedStateProvider类，并设置给SavedStateHandle
* 为了在用户返回时恢复文件数据，需要从SavedStateHandle中获取键为temp_file的Bundle对象


```kotlin
private fun File.saveTempFile() = bundleOf("path", absolutePath)

private fun Bundle.restoreTempFile() = if (containsKey("path")) {
    File(getString("path"))
} else {
    null
}

class TempFileViewModel(savedStateHandle: SavedStateHandle) : ViewModel() {
    private var tempFile: File? = null
    init {
        val tempFileBundle = savedStateHandle.get<Bundle>("temp_file")
        if (tempFileBundle != null) {
            tempFile = tempFileBundle.restoreTempFile()
        }
        savedStateHandle.setSavedStateProvider("temp_file") { // saveState()
            if (tempFile != null) {
                tempFile.saveTempFile()
            } else {
                Bundle()
            }
        }
    }

    fun createOrGetTempFile(): File {
      return tempFile ?: File.createTempFile("temp", null).also {
          tempFile = it
      }
    }
}
```