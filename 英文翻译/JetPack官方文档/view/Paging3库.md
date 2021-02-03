

# Paging3库

* 原始文档：https://developer.android.com/topic/libraries/architecture/paging/v3-overview

[TOC]


## 简介

### 概述
* 该库可以帮助你加载、分页显示来自一个大数据库的数据，数据可以来自于本地、网络
* 该库允许你更有效率的使用网络带宽和系统资源
* 该库可以和其他的JetPac库整合使用


### 优点
* 分页数据的内存缓存
* 内建的请求去重
* 可配置的RecyclerView适配器
* 支持Kotlin协程和Flow
* 内建的错误处理，包括刷新和重试功能


### 安装
```groovy
dependencies {
  def paging_version = "3.0.0-alpha12"

  implementation "androidx.paging:paging-runtime:$paging_version"

  testImplementation "androidx.paging:paging-common:$paging_version"

  // optional - RxJava2 support
  implementation "androidx.paging:paging-rxjava2:$paging_version"

  // optional - RxJava3 support
  implementation "androidx.paging:paging-rxjava3:$paging_version"

  // optional - Guava ListenableFuture support
  implementation "androidx.paging:paging-guava:$paging_version"

  // Jetpack Compose Integration
  implementation "androidx.paging:paging-compose:1.0.0-alpha05"
}
```


### 架构
**库的组件分布在应用的三个层上**
* 持久化层
* ViewModel层
* 界面层

#### 图示
![](https://gitee.com/cc12703/figurebed/raw/master/img/20210114100304.png)

#### PagingSource
* 该组件是位于持久化层的第一个组件
* 每个PagingSource对象都定义了一个数据源、如何从源中获取数据的方法
* 一个PagingSource对象可以从任何一个单一源中加载数据：网络源、本地数据库

#### RemoteMediator
* 该组件是持久化层上的另一个组件
* 该组件处理组合数据源上的数据分页：带本地数据库缓存的网络源


#### Pager
* 该组件位于ViewModel层
* 该组件提供公共接口用于创建PagingData实例
* 该组件需要PagingSource对象和PagingConfig对象

#### PagingData
* 该组件用于连接ViewModel层和界面层
* 该组件是一个容器，用于放置已被分页数据的快照
* 该组件从PagingSource中查询数据，并保存结果


#### PagingDataAdapter
* 该组件是位于界面层的第一个组件
* 该组件是一个RecyclerView适配器，处理已被分页的数据
* 你也可以使用AsyncPagingDataDiffer组件来构建自定义的适配器




## 加载和显示

### 定义数据源
#### 概述
* 第一步是定义一个PagingSource的实现类，用来标识该数据源
* PagingSource类中包含一个load()方法，必须要重载该方法来从对应源中获取数据
* 可以直接使用Kotlin协程来实现异步加载，也支持其他异步框架
    * 对于RxJava，可以实现RxPagingSource
    * 对于ListenableFuture，可以实现ListenableFuturePagingSource

#### 选择键和值的类型
* PagingSource<Key, Value> 有两个类型参数
    * Key定义了用于加载数据的标识类型
    * Value定义了数据本身的类型
* 例子：如果从网络源加载User对象的分页数据，并传入整型的页码给Retrofit库
    * 可以给Key选择Int类型
    * 可以给Value选择Uer类型


#### 创建PagingSource
* 典型的实现中，load()方法会使用构造函数中传入的参数，来加载指定的数据
* 示例中的参数包括
    * backend: 一个后端服务的实例，用来提供数据
    * query: 一个搜索查询，用于通过backend发送给服务器

##### LoadParams
* 包含了加载操作的信息：键、要加载的个数

##### LoadResult
* 包含了加载操作的结果
* 是一个封闭的类，有两种形式
    * 如果加载成功，返回LoadResult.Page对象
    * 如果加载失败，返回LoadResult.Error对象

##### 示例
```kotlin
class ExamplePagingSource(
    val backend: ExampleBackendService,
    val query: String
) : PagingSource<Int, User>() {
  override suspend fun load(
    params: LoadParams<Int>
  ): LoadResult<Int, User> {
    try {
      // 若未定义，则从第一页开始获取
      val nextPageNumber = params.key ?: 1
      val response = backend.searchUsers(query, nextPageNumber)
      return LoadResult.Page(
        data = response.users,
        prevKey = null, // Only paging forward.
        nextKey = response.nextPageNumber
      )
    } catch (e: Exception) {
        // 这里用于处理错误
        // 如果发生一个预期错误，则返回一个LoadResult.Error
    }
  }
}
```

##### load的键更新
![](https://gitee.com/cc12703/figurebed/raw/master/img/20210114105728.png)


#### 错误处理
* 加载数据的请求可能会因为多种原因而失败，特别是通过网络进行加载时
* 在加载过程中上报错误都会遇到过：从load方法中返回LoadResult.Error对象

```kotlin
catch (e: IOException) {
  // IOException 发生网络错误
  return LoadResult.Error(e)
} catch (e: HttpException) {
  // HttpException for any non-2xx HTTP status codes.
  // HttpException 返回了非2xx的HTTP状态码
  return LoadResult.Error(e)
}
```


### 建立数据流
* 需要从PagingSource实现类中建立一个分页数据的数据流。
* 典型的，你需要在ViewModel中建立数据流
* Pager类提供了方法可以暴露出一条PagingData的响应式流，从PagingSource中
* Paging库也支持其他多种流类型：Flow, LiveData，RxJava中的Flowable, Observable

#### 例子
```kotlin
val flow = Pager(
  // 通过给PagingConfig传入附加属性，可以配置数据如何被加载
  PagingConfig(pageSize = 20)
) {
  ExamplePagingSource(backend, query)
}.flow
  .cachedIn(viewModelScope)
```
说明
* 创建Pager实例时，需要一个pagingConfig实例和PagingSource实现实例
* cachedIn()可以使数据流变成可共享的，在提供的CoroutineScope中缓存已加载的数据
* Pager对象可以调用pagingSource对象的load()接口


### 定义RecyclerView适配器
* 你需要设置一个适配器来接收数据到你的RecyclerView中
* Paging库提供了PagingDataAdapter类来实现

#### 例子
```kotlin
class UserAdapter(diffCallback: DiffUtil.ItemCallback<User>) :
  PagingDataAdapter<User, UserViewHolder>(diffCallback) {
  override fun onCreateViewHolder(
    parent: ViewGroup,
    viewType: Int
  ): UserViewHolder {
    return UserViewHolder(parent)
  }

  override fun onBindViewHolder(holder: UserViewHolder, position: Int) {
    val item = getItem(position)
    // 注意这个item可能会有Null，ViewHolder必须要能绑带一个null的item
    holder.bind(item)
  }
}

object UserComparator : DiffUtil.ItemCallback<User>() {
  override fun areItemsTheSame(oldItem: User, newItem: User): Boolean {
    return oldItem.id == newItem.id
  }

  override fun areContentsTheSame(oldItem: User, newItem: User): Boolean {
    return oldItem == newItem
  }
}
```
说明
* UserViewHolder 作为一个视图holder
* 适配器必须定义onCreateViewHolder(), onBindViewHolder()，以及一个DiffUtil.ItemCallback


### 界面上显示数据
#### 操作步骤
1. 创建一个PagingDataAdapter实例
1. 给RecyclerView传入该适配器实例
1. 监控PagingData流，将传入的参数传给适配器的submitData()方法

```kotlin
val viewModel by viewModels<ExampleViewModel>()

val pagingAdapter = UserAdapter(UserComparator)
val recyclerView = findViewById<RecyclerView>(R.id.recycler_view)
recyclerView.adapter = pagingAdapter

// Activities 可以直接使用 lifecycleScope，
// 但是fragments需要使用viewLifecycleOwner.lifecycleScope.
lifecycleScope.launch {
  viewModel.flow.collectLatest { pagingData ->
    pagingAdapter.submitData(pagingData)
  }
}
```
说明
* 该操作需要在activity的onCreate()，fragment的onViewCreated()中进行


### 显示加载状态
#### 概述
* Paging库通过LoadState对象导出了加载状态，用于在界面上使用
* LoadState有三种形式，取决于当前的加载状态

##### 状态
* LoadState.NotLoading 加载操作未被激活、没有错误
* LoadState.Loading 加载操作已经被激活
* LoadState.Error 发生一个错误


#### 使用监听器 -- addLoadStateListener()

```kotlin
lifecycleScope.launch {
  pagingAdapter.loadStateFlow.collectLatest { loadStates ->
    progressBar.isVisible = loadStates.refresh is LoadState.Loading
    retry.isVisible = loadState.refresh !is LoadState.Loading
    errorMsg.isVisible = loadState.refresh is LoadState.Error
  }
}
```

#### 使用适配器 -- LoadStateAdapter
##### 步骤
1. 创建LoadStateAdapter的实现类
1. 调用PagingDataAdapter的withLoadStateHeaderAndFooter()方法

```kotlin
class LoadStateViewHolder(
  parent: ViewGroup,
  retry: () -> Unit
) : RecyclerView.ViewHolder(
  LayoutInflater.from(parent.context)
    .inflate(R.layout.load_state_item, parent, false)
) {
  private val binding = LoadStateItemBinding.bind(itemView)
  private val progressBar: ProgressBar = binding.progressBar
  private val errorMsg: TextView = binding.errorMsg
  private val retry: Button = binding.retryButton
    .also {
      it.setOnClickListener { retry() }
    }

  fun bind(loadState: LoadState) {
    if (loadState is LoadState.Error) {
      errorMsg.text = loadState.error.localizedMessage
    }

    progressBar.isVisible = loadState is LoadState.Loading
    retry.isVisible = loadState is LoadState.Error
    errorMsg.isVisible = loadState is LoadState.Error
  }
}

// 适配器在状态等于LoadState.Loading时，显示一个加载动画
// 在状态等于LoadState.Error时，显示错误信息，重试按键
class ExampleLoadStateAdapter(
  private val retry: () -> Unit
) : LoadStateAdapter<LoadStateViewHolder>() {

  override fun onCreateViewHolder(
    parent: ViewGroup,
    loadState: LoadState
  ) = LoadStateViewHolder(parent, retry)

  override fun onBindViewHolder(
    holder: LoadStateViewHolder,
    loadState: LoadState
  ) = holder.bind(loadState)
}

pagingAdapter
  .withLoadStateHeaderAndFooter(
    header = ExampleLoadStateAdapter(adapter::retry),
    footer = ExampleLoadStateAdapter(adapter::retry)
  )
```



## 来源于网络和数据库

### 概述
* 可以通过保证在用户离线或者网络不稳定时，应用依然可以使用，来提升用户体验
* 其中一种方法就是数据页同时获取自网络和本地数据库
* 这样应用可以直接从本地数据库缓存中加载数据，只在数据库中没有数据可用时向网络请求数据
* Paging库提供了RemoteMediator组件来实现该功能
  * 可以管理从网络加载数据到本地数据库的流程


### 基本使用
#### 概述

![](https://gitee.com/cc12703/figurebed/raw/master/img/20210115113703.png)

* 设想一下，应用从一个网络数据源加载User数据项到本地缓存，存储进Room数据库
* RemoteMediator实现了从网络加载数据并存入数据库的操作，但是并不会直接加载数据到界面
* 应用使用了数据库作为**唯一信任源**，应用只显示缓存在数据库中的数据
* PagingSource实现实例用于从数据库中加载数据并在界面显示

#### 创建Room实体
* 使用Room库创建一个数据库
* 定义一个Room实体， id字段作为主键
* 为这个实体定义一个DAO，包括以下方法
  * insertAll()
  * clearAll()
  * 一个查询方法，使用查询字符串

```kotlin
@Entity(tableName = "users")
data class User(val id: String, val label: String)

@Dao
interface UserDao {
  @Insert(onConflict = OnConflictStrategy.REPLACE)
  suspend fun insertAll(users: List<User>)

  @Query("SELECT * FROM users WHERE label LIKE :query")
  fun pagingSource(query: String): PagingSource<Int, User>

  @Query("DELETE FROM users")
  suspend fun clearAll()
}
```

#### 实现RemoteMediator
##### 构建参数
* query 查询字符串，用于从后端服务中获取数据
* database Room数据库实例
* networkService 后端服务器的接口实例
* Key, Value数据类型需要和在定义PagingSource时一样的类型

##### load方法
###### 功能
* 更新支持数据集
* 无效PagingSource

###### 参数
* PagingState 包含页加载所需要的信息
* LoadType 加载类型：刷新(REFRESH)，后部追加(APPEND)，前部追加(PREPEND)

###### 返回值
* 一个MediatorResult对象
  * MediatorResult.Error 包含一个错误描述信息
  * MediatorResult.Success 包含一个标记，表明是否有数据在加载

###### 流程
1. 决定需要从网络加载哪些页（取决于加载类型、目前为止已加载的数据）
1. 触发一次网络请求
1. 根据加载结果执行以下动作
    * 如果加载成功且数据不为空：保存数据到数据库，并返回 MediatorResult.Success(endOfPaginationReached = false)
    * 如果加载成功且数据为空：返回MediatorResult.Success(endOfPaginationReached = true)
    * 如果请求失败：返回MediatorResult.Error

#### 创建Pager对象
* 和从单一网络源创建Pager对象很相似，除了以下不同点
    * 不直接传入PagingSource对象，需要从DAO中使用查询函数获取
    * 需要传入RemoteMediator实现实例

```kotlin
val userDao = database.userDao()
val pager = Pager(
  config = PagingConfig(pageSize = 50)
  remoteMediator = ExampleRemoteMediator(query, database, networkService)
) {
  userDao.pagingSource(query)
}
```


### 管理远端键
#### 概述
* 远端键：RemoteMediator实现用来告诉后端服务器下次要加载那些数据
* 最简单情况下，分页数据的每一个数据项都包含一个远端键，你可以很容易的引用
* 如果远端键不和数据项一一对应，则需要在load()中分开存储和管理它们

#### 添加远端键表
* 当远端键不直接和列表项关联时，最好的方式就在独立的本地数据库表中存储它们

```kotlin
@Entity(tableName = "remote_keys")
data class RemoteKey(val label: String, val nextKey: String?)

@Dao
interface RemoteKeyDao {
  @Insert(onConflict = OnConflictStrategy.REPLACE)
  suspend fun insertOrReplace(remoteKey: RemoteKey)

  @Query("SELECT * FROM remote_keys WHERE label = :query")
  suspend fun remoteKeyByQuery(query: String): RemoteKey

  @Query("DELETE FROM remote_keys WHERE label = :query")
  suspend fun deleteByQuery(query: String)
}
```

#### 使用远端键加载
* 当load()方法需要使用远端键时，你必须使用不同的方式来定义它
* 不同点
  * 使用额外属性引用远端键表的DAO
  * 通过查询远端键表来决定使用哪个键用于下次加载
  * 插入、保存从网络数据源返回的远端键，附加在分页数据中

```kotlin
@OptIn(ExperimentalPagingApi::class)
class ExampleRemoteMediator(
  private val query: String,
  private val database: RoomDb,
  private val networkService: ExampleBackendService
) : RemoteMediator<Int, User>() {
  val userDao = database.userDao()
  val remoteKeyDao = database.remoteKeyDao()

  override suspend fun load(
    loadType: LoadType,
    state: PagingState<Int, User>
  ): MediatorResult {
    return try {
      // 网络load方法有一个可选的字符串参数。
      // 对于第一页的每一页，可以传入一个从上一页返回的字符串标识，让该页从脱离的地方继续下去。
      // 对于REFRESH，传入null参数来获取第一页
      val loadKey = when (loadType) {
        LoadType.REFRESH -> null
        // In this example, you never need to prepend, since REFRESH
        // will always load the first page in the list. Immediately
        // return, reporting end of pagination.
        // 在这个例子中，你不需要PREPEND，因为REFRESH会一直加载列表中的第一页。立刻返回，并上报分页结束
        LoadType.PREPEND -> return MediatorResult.Success(
          endOfPaginationReached = true
        )
        // 为下一次的远程将查询remoteKeyDao
        LoadType.APPEND -> {
          val remoteKey = database.withTransaction {
            remoteKeyDao.remoteKeyByQuery(query)
          }

          // 当追加时，你必须明确的检查该页键是否为null，因为null只有首次加载时才有效。
          // 如果你在APPEND时接收到了null，那意味着你接触到了分页的最后，已经没有多余项可以加载了
          if (remoteKey.nextKey == null) {
            return MediatorResult.Success(
              endOfPaginationReached = true
            )
          }

          remoteKey.nextKey
        }
      }

      // 如果使用Retrofit来进行网络加载，就不需要使用withContext(Dispatcher.IO)块来包装代码了。因为Retrofit的协程调度适配器会在工作线程中进行分发
      val response = networkService.searchUsers(query, loadKey)

      // 在事务中存储已加载数据和下一个键，所以数据一直都是一致的
      database.withTransaction {
        if (loadType == LoadType.REFRESH) {
          remoteKeyDao.deleteByQuery(query)
          userDao.deleteByQuery(query)
        }

        // 更新远端键
        remoteKeyDao.insertOrReplace(
          RemoteKey(query, response.nextKey)
        )

        // 插入所有新用户到数据库，无效当前的PagingData，允许分页数据更新到数据库
        userDao.insertAll(response.users)
      }

      MediatorResult.Success(
        endOfPaginationReached = response.nextKey == null
      )
    } catch (e: IOException) {
      MediatorResult.Error(e)
    } catch (e: HttpException) {
      MediatorResult.Error(e)
    }
  }
}
```


### 本地刷新
* 如果你的应用只支持从列表头部进行网络刷新，RemoteMediator就不需要定义追加加载行为了。
* 如果你的应用支持增量从网络加载数据到本地数据库，你必须支持该功能：在锚点处（用户滚动位置）恢复分页开始点。
* Room的PagingSource实现已经处理了这些。但是如果你不使用Room，则可以覆盖PagingSource.getRefreshKey()方法来处理这些

```kotlin
// Item-keyed.
override fun getRefreshKey(state: PagingState): String? {
  return state.anchorPosition?.let { anchorPosition ->
    state.getClosestItemToPosition(anchorPosition)?.id
  }
}

// Positional.
override fun getRefreshKey(state: PagingState): Int? {
  return state.anchorPosition
}
```

#### 图示
展示了开始从数据库加载，当数据库数据过期时，从网络加载
![](https://gitee.com/cc12703/figurebed/raw/master/img/20210117001821.png)





## 变换数据流

### 概述
* 当你开始使用分页数据时，你经常需要在加载数据流后变换它们
* 例子：过滤列表中的数据项，在界面显示前转换数据项为不同的类型，添加列表分隔符


### 基本变换
* 因为PagingData是包装在响应流中的，你可以在数据加载和界面显示上增量的应用变换操作
* 为了给每个PagingData对象应用变换，可以将变换操作放置在流的map操作中

#### 例子1
```kotlin
pager.flow // 数据类型是 Flow<PagingData<User>>.
  // map外部流：变换会应用给每一个新生成的PagingData
  .map { pagingData ->
    // 在代码块中的变换会应用给分页数据中的每个项
}
```

#### 例子2
```kotlin
pager.flow 
    .map { pagingData ->
        pagingData.filter { user -> !user.hiddenFromUi }
            .map { user -> UiModel(user) }
    }
    .cachedIn(viewModelScope)
```
说明
* 外部的map用于数据流的，代码块中的操作会应用于流中的每个PagingData对象
* filter操作会丢弃掉任何界面上不显示的列表项
* 另一个map操作从列表中将每个User对象转换成UiModel类型
* cachedIn()会缓存在它之前的任何变换的结果，所以该函数是在ViewModel中最后调用的


### 添加列表分隔符
* Paging库支持动态列表分隔符
* 可以通过直接向数据流（像RecyclerView）插入分隔符，来提升列表的可读性
* 分隔符就像一个全功能的ViewHolder对象，可以和用户交互、辅助获取焦点、提供View的所有特性

#### 引入步骤
1. 调整界面模型使适应分隔项
1. 转换数据流用于动态添加分隔符
1. 更新界面来处理分隔符项

#### 调整界面模型
* Paging库以实际的列表项来插入列表分隔符到RecyclerView
* 分隔符项必须和列表中的其他项可区分，因为分隔界面和其他列表项在界面上是不同的
* 解决方法就是创建Kotin的sealed类和子类来表示你的数据和分隔符
* 或者你创建一个基类，列表项类和分隔符类用改基类派生出来


```kotlin
sealed class UiModel {
  class UserModel(val id: String, val label: String) : UiModel() {
    constructor(user: User) : this(user.id, user.label)
  }

  class SeparatorModel(val description: String) : UiModel()
}
```

#### 转换数据流
* 必须对数据流进行转换，在数据项被加载后显示前
* 转换步骤
  1. 转换列表项以反应出基本项类型
  1. 使用PagingData.insertSeparators()来加入分隔符


```kotlin
pager.flow.map { pagingData: PagingData<User> 
  .map { user ->
    // 转换数据项为 UiModel.UserModel
    UiModel.UserModel(user)
  }
  .insertSeparators<UiModel.UserModel, UiModel> { before, after ->
    when {
      before == null -> UiModel.SeparatorModel("HEADER")
      after == null -> UiModel.SeparatorModel("FOOTER")
      shouldSeparate(before, after) -> UiModel.SeparatorModel(
        "BETWEEN ITEMS $before AND $after"
      )
      // 返回Null避免只在两个数据项之间添加分隔符
      else -> null
    }
  }
}
```

#### 界面中处理分隔符
* 最后一个步骤就是改变你的界面以适应分隔符类型
* 创建布局和分隔符项的view-holder
* 修改列表适配器使用RecyclerView.ViewHolder作为view-holder类型（可以处理多个view-holder类型）
* 变更步骤
  1. 在onCreateViewHolder()、onBindViewHolder()中增加类型判断，来处理列表分隔符
  1. 实现一个比较操作符

```kotlin
class UiModelAdapter :
  PagingDataAdapter<UiModel, RecyclerView.ViewHolder>(UiModelComparator) {

  override fun onCreateViewHolder(
    parent: ViewGroup,
    viewType: Int
  ) = when (viewType) {
    R.layout.item -> UserModelViewHolder(parent)
    else -> SeparatorModelViewHolder(parent)
  }

  override fun getItemViewType(position: Int) = when (getItem(position)) {
    is UiModel.UserModel -> R.layout.item
    is UiModel.SeparatorModel -> R.layout.separator_item
    null -> throw IllegalStateException("Unknown view")
  }

  override fun onBindViewHolder(
    holder: RecyclerView.ViewHolder,
    position: Int
  ) {
    val item = getItem(position)
    if (holder is UserModelViewHolder) {
      holder.bind(item as UserModel)
    } else if (holder is SeparatorModelViewHolder) {
      holder.bind(item as SeparatorModel)
    }
  }
}

object UiModelComparator : DiffUtil.ItemCallback<UiModel>() {
  override fun areItemsTheSame(
    oldItem: UiModel,
    newItem: UiModel
  ): Boolean {
    val isSameRepoItem = oldItem is UiModel.UserModel
      && newItem is UiModel.UserModel
      && oldItem.id == newItem.id

    val isSameSeparatorItem = oldItem is UiModel.SeparatorModel
      && newItem is UiModel.SeparatorModel
      && oldItem.description == newItem.description

    return isSameRepoItem || isSameSeparatorItem
  }

  override fun areContentsTheSame(
    oldItem: UiModel,
    newItem: UiModel
  ) = oldItem == newItem
}
```