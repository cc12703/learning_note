

# RecyclerView

* 原始文档：https://developer.android.com/guide/topics/ui/layout/recyclerview
* 原始文档：https://developer.android.com/guide/topics/ui/layout/recyclerview-custom

## 基础用法

### 概述
* RecyclerView可以使有效显示大型数据集更简单
* 你可以指定数据，定义每个项的界面，库会在需要显示它们时动态的创建界面元素

#### 回收项
* RecyclerView会回收独立的项
* 当一个项滚动出屏幕后，RecyclerView不会销毁它的视图，会在新的项上重用这个视图
* 视图重用大大的提高了应用性能、降低了能源消耗


### 关键类
#### RecyclerView
* 是一个ViewGroup，包含了数据对应的视图
* 它本身也是一个视图，可以用像其他界面元素一样的方式，添加到布局中

#### 视图持有者
* 对应与列表中每一个独立的项
* 当视图持有者对象被创建后，需要RecyclerView绑定数据到该对象
* 通过扩展RecyclerView.ViewHolder来定义视图持有者

#### 适配器
* RecyclerView需要调用适配器来请求视图，将数据绑定到视图上
* 通过扩展RecyclerView.Adapter来定义适配器

#### 布局管理器
* 布局管理器用于布置列表中的独立元素
* 可以使用RecyclerView中预置的管理器，或者自定义一个
* 通过继承LayoutManager类来创建一个布局管理器


### 实现步骤
1. 需要确定界面是列表形式的还是表格形式的
    * 通常来说，你可以使用RecyclerView库中标准布局管理器的其中一个
1. 设计每个列表项的样子和行为
    * 基于设计，扩展ViewHolder类
    * ViewHolder提供了列表项的所有功能
    * ViewHolder包装了视图，该视图由RecyclerView进行管理
1. 定义一个适配器，用于分配给ViewHolder分配数据


### 管理布局
* RecyclerView中的项是通过layoutManager进行排列的
* 库提供了几个预置的类

#### LinearLayoutManager
* 将项组织成一维的列表

#### GridLayoutManager
* 将项组织成二维的表格
* 如果表格是垂直组织的，管理器会让每个行有相同的宽度和高度，但是不同行会有不同的高度
* 如果表格是水平组织的，管理器会让每个列有相同的宽度和高度，但是不同列会有不同的宽度

#### StaggeredGridLayoutManager
* 类似于GridLayoutManager，但是并不要求每行有相同的高度或者每列有相同的宽度
* 每行、每列的项可以和其他项有偏移


### 实现适配器和视图持有者
* 当确定好布局后，就需要实现适配器和视图持有者
* 这两个类需要协调工作，来将数据显示出来
* ViewHolder是视图的一个包装器，该视图包含了独立项的布局信息
* Adapter按需创建ViewHolder对象，给视图设置数据（该过程称为绑定）


#### 适配器的关键方法
* onCreateViewHolder() 当需要创建新ViewHolder时被调用
    * 该方法创建、初始化ViewHolder对象，并为其分配一个视图
    * 但是并不为该视图填充数据
* onBindViewHolder() 当需要给ViewHolder关联数据时被调用
    * 该方法获取相关的数据，并填充给ViewHolder中的布局
* getItemCount() 当需要获取数据集大小时被调用
    * 例子：通讯录应用，会有通讯录总数。RecyclerView使用该值来确定是否有更多项需要显示


#### 例子
```kotlin
class CustomAdapter(private val dataSet: Array<String>) :
        RecyclerView.Adapter<CustomAdapter.ViewHolder>() {

    /**
     * 提供一个所使用视图的类型的引用
     * (custom ViewHolder).
     */
    class ViewHolder(view: View) : RecyclerView.ViewHolder(view) {
        val textView: TextView

        init {
            // 给ViewHolder视图定义点击监听器
            textView = view.findViewById(R.id.textView)
        }
    }

    // 创建新视图(由布局管理器触发)
    override fun onCreateViewHolder(viewGroup: ViewGroup, viewType: Int): ViewHolder {
        // 创建一个新视图，给列表项定义界面
        val view = LayoutInflater.from(viewGroup.context)
                .inflate(R.layout.text_row_item, viewGroup, false)

        return ViewHolder(view)
    }

    // 替换视图的内容(由布局管理器触发)
    override fun onBindViewHolder(viewHolder: ViewHolder, position: Int) {
        // 获取对应位置的数据，并替换掉视图的内容
        viewHolder.textView.text = dataSet[position]
    }

    // 返回数据集的大小(由布局管理器触发)
    override fun getItemCount() = dataSet.size

}
```

text_row_item.xml的内容
```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="@dimen/list_item_height"
    android:layout_marginLeft="@dimen/margin_medium"
    android:layout_marginRight="@dimen/margin_medium"
    android:gravity="center_vertical">

    <TextView
        android:id="@+id/textView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/element_text"/>
</FrameLayout>
```


## 高级用法

### 修改布局
* RecyclerView使用布局管理器在屏幕上放置独立项，当它们不可见时重用它们的视图
* 为了重用视图，布局管理器需要是适配器使用数据集中不同的数据来替换掉视图的内容
* 重用视图可以通过避免创建不需要的视图或执行昂贵的findViewById()操作而提升性能

#### 标准布局管理器
* LinearLayoutManager : 以一维列表的方式组织独立项，提供类似于ListView布局的功能
* GridLayoutManager ：以二维表格的方式组织独立项，提供类似于GridView布局的功能
* StaggeredGridLayoutManager : 以二维表格方式组织独立项，每列相对于前一列会有一些偏移


### 添加项动画
* 无论何时独立项改变了，RecyclerView会使用动画来改变独立项的展示
* 动画是一个扩展与RecyclerView.ItemAnimator的对象
* 默认情况下，RecyclerView使用DefaultItemAnimator来提供动画
* 如果需要自定义，可以通过扩展ItemAnimator来创建自己的动画类


### 启用独立项选择
* recyclerview-selection库可以允许用户通过触摸或鼠标来选择列表中的项
* 你可以控制被选择的项要如何显示
* 你也可以控制选择行为本身的策略：有哪些项可以被选择，有多少项可以被选择

#### 增加步骤
1. 确定使用哪种类型的选择键，然后构建ItemKeyProvider对象
    * 有三种类型的键：Parcelable, String, Long
1. 实现ItemDetailsLookup
    * 该类可以使**选择库**获取发出MotionEvent事件的RecyclerView项的信息
    * 该类实际上就是ItemDetail实例的工厂类，通过扩展ViewHolder实体
1. 更新RecyclerView中独立项的视图，来响应用户选择或取消选择的情况
    * **选择库**对选择项不提供默认的展示样式，需要通过onBindViewHolder()来提供
    * 推荐方法
        1. 在onBindViewHolder()中，调用视图的setActivated()\setSelected()来设置true\false
        1. 更新视图的样式来呈现出激活状态
1. 使用ActionMode为用户提供工具，在选择项后执行指定操作
    * 注册一个SelectionTracker.SelectionObserver，在选择项改变后被通知到
    * 当选择被首次创建时，启动ActionMode向用户展现一个菜单，会提供特定的动作
    * 例子:在ActionMode栏中增加删除按键，连接回头箭头到栏中来清除选择
1. 执行解释型的二次动作
    * 在事件处理流水线的结尾时，库会确定：用户是试着点击项来激活它、试着拖拽它、选择多个项
    * 通过注册一个适当的监听器可以响应这些解释信息
1. 使用SelectionTracker.Builder封装所有逻辑
    * 为了构建SelectionTracker实例，你必须提供给Builder一个和初始RecyclerView时同一个适配器对象
    * 最好在适配器中，在适配器被创建后注入SelectionTracker实例
    ```kotlin
    var tracker = SelectionTracker.Builder(
        "my-selection-id",
        recyclerView,
        StableIdKeyProvider(recyclerView),
        MyDetailsLookup(recyclerView),
        StorageStrategy.createLongStorage())
            .withOnItemActivatedListener(myItemActivatedListener)
            .build()
    ```
1. 在activity生命周期事件中包含选择信息
    * 为了跨生命周期事件而保存选择信息，需要在activity的onXXXInstanceState()中调用选择跟踪器的onXXXInstanceState()
    * 必须给SelectionTracker.Builder构建方法提供一个唯一的选择ID（在Saved State中标识数据）