

# Paging

* 原始文档：https://developer.android.com/topic/libraries/architecture/paging

[TOC]

## 简介

### 概述
* Paging库帮助你一次加载、显示一小块数据
* 按需加载部分数据，以降低对网络带宽和系统资源的占用


### 安装
```groovy
dependencies {
  def paging_version = "2.1.2"

  implementation "androidx.paging:paging-runtime:$paging_version" 
  // Kotlin 使用 paging-runtime-ktx

  
  testImplementation "androidx.paging:paging-common:$paging_version" 
  // Kotlin 使用 paging-common-ktx

  // 可选 - 支持 RxJava 
  implementation "androidx.paging:paging-rxjava2:$paging_version" 
  // Kotlin 使用 paging-rxjava2-ktx
}
```

### 库架构
**描述Pagin库的主要组件

#### PagedList
* PagedList是关键组件，用于分块、分页加载应用数据
* 当需要更多数据时，数据会被分页进行PagedList对象
* 只要加载的数据发送变化，一个新的PagedList实例会被发送到LiveData的观测者
* 随着PagedList对象生成，应用界面显示其他的内容，这些都会响应界面控制器的生命周期

```kotlin
class ConcertViewModel(concertDao: ConcertDao) : ViewModel() {
    val concertList: LiveData<PagedList<Concert>> =
            concertDao.concertsByDate().toLiveData(pageSize = 50)
}
```

#### 数据
* 每个PagedList实例从对应的DataSource中加载应用最新数据的快照
*  数据流从应用后端、数据库进入PagedList对象

```kotlin
@Dao
interface ConcertDao {
    // 整型参数告诉Room使用PositionalDataSource对象
    @Query("SELECT * FROM concerts ORDER BY date DESC")
    fun concertsByDate(): DataSource.Factory<Int, Concert>
}
```

#### 界面
* PagedList使用PagedListAdapter加载数据项到RecyclerView
* 这些类在一起协同工作：获取显示已加载的内容，预取还未显示的内容，内容变更时增加动画


### 支持不同的数据架构
#### 支持的架构
* 从后端服务器获取数据
* 从本地数据库获取数据
* 整合不同数据源，使用本地数据库作为缓存

#### 示意图
![](https://gitee.com/cc12703/figurebed/raw/master/img/20210112102148.png)

##### 只使用网络
* 从后端服务器获取数据直接显示
* 显示异步版本的Retrofit接口
* 加载数据到自定义的DataSource对象

###### 注意
* Paging库的DataSource对象不进行任何的错误处理，因为不同应用显示错误界面的方式都不一样
* 如果有错误发生，遵从你的结果回调，过一段时候后重新进行请求


##### 只使用数据库
* 使RecyclerView观察本地数据库，最好使用Room库
* 这种方式，只要数据库的数据被插入、修改，这些变更会自动在RecyclerView中反应，并显示在界面上


##### 使用网络和数据库
* 在观察本地数据库后，你可以使用PagedList.BoundaryCallback来监听何时数据库数据过期
* 你可以从网络获取更多数据，插入到数据库中
* 如果你的界面观察着数据库，界面就会自动更新


### 处理网络错误
* 有一个点很重要，不要将网络看成是一直有效、一直无效的。很多网络都是不稳定的
    * 特定服务器会失效，而不响应网络请求
    * 设备连接的网络可能是慢的、脆弱的

#### 处理方法
* 对每个请求检查是否失败
* 当网络不可用时，尽量优雅的进行恢复
    * 如果在数据刷新时网络出错，可以显示一个“重试”按键给用户选择
    * 如果在分页阶段网络出错，可以自动进行重新请求


### 升级已有的应用

#### 自定义分页逻辑
* 如果原应用是自定义逻辑完成加载分页数据的话，你可以直接使用PagedList来替换原实现
* PagedList实例提供内建功能来连接常用的数据源
* PagedList也提供适配器来适配RecyclerView

#### 使用列表加载数据
* 如果原应用使用列表容器来保存后端数据，可以直接使用PagedList替换
* LiveData\<PagedList\>或Obseravle\<PagedList\>可以传递数据变更给界面
* PagedList可以最小化加载时间、内存占用
* 使用PagedList不需要修改原应用的界面结构和数据更新逻辑

#### 使用数据游标关联列表控件
* 使用CursorAdapter来关联Cursor中的数据和ListView控件，需要做如下修改
    * 从ListView迁移到RecyclerView
    * 使用Room或PositionDataSource来替换Cursor组件

//TODO:


#### 使用AsyncListUtil
* 如果你使用AsyncListUtil来异步加载、显示分组的数据，Paging库可以带来如下优点
    * 数据不再需要被定位：Paging库可以让你直接从后端加载数据
    * 数据不再会变得很多很多：Paging库可以让你将数据分页，直到没有数据存在
    * 观察数据会更简单：Paging库可以保持你的数据，应用的ViewModel可以保存一个可观察的数据结构



### 例子
#### 使用LivData观察分页数据
* 随着concert事件在数据库中被添加、移除、修改。
* RecyclerView会自定义和有效的被更新

```kotlin
@Dao
interface ConcertDao {
    // 整型类型参数告知Room使用PositionalDataSource对象（基于位置加载数据）
    @Query("SELECT * FROM concerts ORDER BY date DESC")
    fun concertsByDate(): DataSource.Factory<Int, Concert>
}

class ConcertViewModel(concertDao: ConcertDao) : ViewModel() {
    val concertList: LiveData<PagedList<Concert>> =
            concertDao.concertsByDate().toLiveData(pageSize = 50)
}

class ConcertActivity : AppCompatActivity() {
    public override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val viewModel: ConcertViewModel by viewModels()
        val recyclerView = findViewById(R.id.concert_list)
        val adapter = ConcertAdapter()
        viewModel.concertList.observe(this, PagedList(adapter::submitList))
        recyclerView.setAdapter(adapter)
    }
}

class ConcertAdapter() :
        PagedListAdapter<Concert, ConcertViewHolder>(DIFF_CALLBACK) {
    fun onBindViewHolder(holder: ConcertViewHolder, position: Int) {
        val concert: Concert? = getItem(position)

        // 注意：concert是一个占位符
        holder.bindTo(concert)
    }

    companion object {
        private val DIFF_CALLBACK = object :
                DiffUtil.ItemCallback<Concert>() {
            // Concert 详情信息在数据库重加载后可能会变更
            // 但是ID是固定不变的
            override fun areItemsTheSame(oldConcert: Concert,
                    newConcert: Concert) = oldConcert.id == newConcert.id

            override fun areContentsTheSame(oldConcert: Concert,
                    newConcert: Concert) = oldConcert == newConcert
        }
    }
}
```

#### 使用RxJava2观察分页数据

```kotlin
class ConcertViewModel(concertDao: ConcertDao) : ViewModel() {
    val concertList: Observable<PagedList<Concert>> =
            concertDao.concertsByDate().toObservable(pageSize = 50)
}

class ConcertActivity : AppCompatActivity() {
    private val adapter = ConcertAdapter()
    private val viewModel: ConcertViewModel by viewModels()

    private val disposable = CompositeDisposable()

    public override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        val recyclerView = findViewById(R.id.concert_list)
        recyclerView.setAdapter(adapter)
    }

    override fun onStart() {
        super.onStart()
        disposable.add(viewModel.concertList
                .subscribe(adapter::submitList)))
    }

    override fun onStop() {
        super.onStop()
        disposable.clear()
    }
}
```



## 显示分页列表

### 连接界面和ViewModel
* 可以将LiveData\<PagedList\>实例连接上PagedListAdapter

```kotlin
class ConcertActivity : AppCompatActivity() {
    private val adapter = ConcertAdapter()
    private val viewModel: ConcertViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState);
        viewModel.concerts.observe(this, Observer { adapter.submitList(it) })
    }
}

class ConcertAdapter() :
        PagedListAdapter<Concert, ConcertViewHolder>(DIFF_CALLBACK) {

    override fun onBindViewHolder(holder: ConcertViewHolder, position: Int) {
        val concert: Concert? = getItem(position)
        holder.bindTo(concert)
    }

    companion object {
        private val DIFF_CALLBACK = ... 
    }
}
```
说明
* 数据源会提供一个PagedList的实例，activity将其发生给适配器
* PageListAdapter：定义如何计算更新，自动处理分页和列表差异
* ViewHolder只需要绑定到特定的数据项中即可
* PageListAdapter会使用PagedList.Callback来处理页加载事件
* 随着用户滑动，PagedListAdapter会调用PagedList.loadAround()来告知PagedList需要从数据源获取哪些数据项


### 实现差异比较回调

```kotlin
private val DIFF_CALLBACK = object : DiffUtil.ItemCallback<Concert>() {
    // ID属性来表明数据项是一样的
    override fun areItemsTheSame(oldItem: Concert, newItem: Concert) =
            oldItem.id == newItem.id

    // 如果使用 == 操作符，需要确保对象实现了.equals()函数
    override fun areContentsTheSame(
            oldItem: Concert, newItem: Concert) = oldItem == newItem
}
```
说明
* 因为适配器包含了你定义的数据项比较方法，所以当新的PagedList对象被加载后，适配器可以自动侦测到数据项的变化
* 适配器可以在RecyclerView中触发更有效率数据项动画


### 在界面提供占位符
* 当你想在应用获取到数据前，就显示列表界面时，你可以给用户展示占位符列表项
* PagedList可以处理这种情况，通过在数据加载前将数据项设置为null

**注意**
* 默认情况下，Paging库会打开占位符功能

#### 优点
##### 支持滚动条
* PagedList会给PagedListAdapter提供数据项的个数
* 这个信息可以让适配器绘制一条滚动条，来展示整个列表的大小
* 当新页被加载后，滚动条不会跳跃（因为列表大小没有变化）

##### 不需要加载中提示
* 因为列表大小已经知道了，所以不需要告知用户还有更多数据在加载中
* 占位符自身已经包含了这些信息

#### 实现前提
##### 需要可计数的数据集
* Room的DataSource可以高效的计数数据项个数
* 如果你使用自定义本地储存或者只从网络获取数据，这时候计算数据集的数量会变得代价高昂和不可能

##### 需要适配器占用未加载项
* 使用适配器、表现层算法会需要列表膨胀来处理null列表项
    * 当绑定数据到ViewHolder时，你需要提供一个默认值用于显示未加载的数据

##### 需要数据项界面大小一样
* 如果列表项的大小会随着他们内容而变化，列表项之间的淡入淡出效果不会很好看
* 我们强烈建议在这种情况下禁止使用placeHolder




## 收集分页数据

### 构建可观测列表
* 典型的，界面可以观察一个LiveData<PagedList>对象（对于RxJava2, 则观察Flowable<PagedList>或者Observable<PagedList>对象），该对象会保存在应用ViewModel中
* 这个可观察对象在表现侧和应用列表数据之间形成了一个连接

##### 构建方法
* 需要将一个DataSource.Factory实例传入LivePagedListBuilder或RxPagedListBuilder对象中
* 一个DataSource对象从单个PagedList中加载分页数据
* 这个工厂类会在内容更新后，创建一个新的PagedList对象。像数据库表无效、网络重置
* Room库有提供一个DataSource.Factor.

```kotlin
//ConcetDao
@Dao
interface ConcertDao {
    @Query("SELECT * FROM concerts ORDER BY date DESC")
    fun concertsByDate(): DataSource.Factory<Int, Concert>
}

//ConcertViewModel
val myConcertDataSource : DataSource.Factory<Int, Concert> =
       concertDao.concertsByDate()

val concertList = myConcertDataSource.toLiveData(pageSize = 50)
```


### 定义分页配置
* 为了让LiveData<PagedList>能用于更复杂情况，你需要定义一个自己的分页配置

#### 属性项
##### 页大小
* 每个页的数据项个数

##### 预取距离
* 给定界面中最后一个可见项，在离最后一个项多少个数时，Paging库会试着提前获取数据
* 这个值应该是比页大小大很多倍

##### 占位符展示
* 确定在列表项还没有加载完成时，界面是否显示占位符


#### 自定义加载逻辑
* 若想在Paging库加载数据时有更多的控制，可以给LivePagedListBuilder传入一个自定义的Executor对象

```kotlin
//ConcetViewModel
val myPagingConfig = Config(
        pageSize = 50,
        prefetchDistance = 150,
        enablePlaceholders = true
)

val myConcertDataSource : DataSource.Factory<Int, Concert> =
        concertDao.concertsByDate()

val concertList = myConcertDataSource.toLiveData(
        pagingConfig = myPagingConfig,
        fetchExecutor = myExecutor
)
```


### 选择正确的数据源

#### PageKeyedDataSource
* 用于页面中内嵌了后一个/前一个的键
* 例子：从网络上获取社交媒体文章时，将其加载到子序列中时，需要传入一个nextPage的标记

#### ItemKeyedDataSource
* 用于使用数据时，需要从第N项获取第N+1项
* 例子：为一个讨论应用，获取内嵌的评论信息，你就需要传入上个评论的ID，以便获取下个评论的内容

#### PositionalDataSource
* 用于从任何位置都可以获取页数据的情况
* 该类支持从你选择的任何位置开始请求一个数据项集合
* 例子：从位置1500开始，获取50个数据项



### 数据无效通知
* 当使用Paging库时，若数据表、数据行无效时，则数据层可以通知应用的其他层
* 可以通过调用DataSource的invalidate()函数来实现这个功能

#### 注意
* 通过使用点击刷新模式，应用的界面也可以触发数据无效化的功能



### 自定义数据源
* 如果你使用自定义的本地数据方案、直接从网络获取数据，你可以实现一个DataSource的子类
* 你可以创建一个DataSource.Factory的具体子类，来把自定义数据加载到PagedList中

```kotlin
class ConcertTimeDataSource() :
        ItemKeyedDataSource<Date, Concert>() {

    override fun getKey(item: Concert) = item.startTime

    override fun loadInitial(
            params: LoadInitialParams<Date>,
            callback: LoadInitialCallback<Concert>) {
        val items = fetchItems(params.requestedInitialKey,
                params.requestedLoadSize)
        callback.onResult(items)
    }

    override fun loadAfter(
            params: LoadParams<Date>,
            callback: LoadCallback<Concert>) {
        val items = fetchItemsAfter(
            date = params.key,
            limit = params.requestedLoadSize)
        callback.onResult(items)
    }
}

class ConcertTimeDataSourceFactory :
        DataSource.Factory<Date, Concert>() {
    val sourceLiveData = MutableLiveData<ConcertTimeDataSource>()
    var latestSource: ConcertDataSource?
    override fun create(): DataSource<Date, Concert> {
        latestSource = ConcertTimeDataSource()
        sourceLiveData.postValue(latestSource)
        return latestSource
    }
}
```


### 数据更新
* 如果你构建了一个可观察的PagedList对象，就需要考虑一下如何进行数据更新
* 如果你直接从Room数据库中加载数据，更新时数据会自动更新到界面
* 当你使用分页的网络接口，典型情况下会有用户交互（像点击刷新）
    * 该操作可以作为一个信号，用于无效掉最近用过的DataSource
    * 然后你需要请求一个新的数据源实例


```kotlin
class ConcertActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        // ...
        concertTimeViewModel.refreshState.observe(this, Observer {
            swipeRefreshLayout.isRefreshing =
                    it == MyNetworkState.LOADING
        })
        swipeRefreshLayout.setOnRefreshListener {
            concertTimeViewModel.invalidateDataSource()
        }
    }
}

class ConcertTimeViewModel(firstConcertStartTime: Date) : ViewModel() {
    val dataSourceFactory = ConcertTimeDataSourceFactory(firstConcertStartTime)
    val concertList: LiveData<PagedList<Concert>> =
            dataSourceFactory.toLiveData(
                pageSize = 50,
                fetchExecutor = myExecutor
            )

    fun invalidateDataSource() =
            dataSourceFactory.sourceLiveData.value?.invalidate()
}
```


### 数据映射
* Paging库提供基于数据项、基于数据页的数据转换
* 这个功能对于想在数据加载后对数据进行包装、转换、准备非常有用
* 由于这个过程是在fetc执行器中完成的，所以你可以做一些比较费时的工作（读取磁盘、查询数据库）

```kotlin
class ConcertViewModel : ViewModel() {
    val concertDescriptions : LiveData<PagedList<String>>
        init {
            val concerts = database.allConcertsFactory()
                    .map { "${it.name} - ${it.date}" }
                    .toLiveData(pageSize = 50)
        }
}
```



