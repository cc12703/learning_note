

# Paging

* 原始文档：https://developer.android.com/topic/libraries/architecture/paging


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