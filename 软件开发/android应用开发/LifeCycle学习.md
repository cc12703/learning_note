

#LifeCycle学习

## 总体

* google开发的生命周期感知框架
* 用于感知activity，fragment, service的生命周期



## 模块

### lifecycle
* 生命周期感知框架
* 包括 owner 和 observer 两部分
* 在owner的生命周期发生变化时，发送信息给observer

### livedata 
* 带生命周期感知功能的数据持有者

### view-model
* 数据管理器，用于activity, fragement的数据管理

### support v4
* 实现lifecycle的owner，即activity, fragment



## lifecycle

### Lifecycle类
* 定义了android的生命周期，包括事件，状态
* 事件：ON_CREATE，ON_START，ON_RESUME，ON_PAUSE，ON_STOP， ON_DESTROY， ON_ANY
* 状态：INITIALIZED，DESTROYED，CREATED，STARTED，RESUMED
* 实现类为 LifecycleRegistry

### LifecycleRegistry类

#### 接口
* addObserver 注册监听器
* removeObserver 注销监听器
* handleLifecycleEvent 处理生命周期事件，由owner调用

### ReportFragment类

#### 功能
* 用于上报生命周期，由SupportActivity在onCreate时创建

#### 逻辑
1. 向fragment-manager注册自己，已接收回调
2. 将回调转发给Activity的LifecycleOwner处理


## liveData

### LiveData类型

#### 功能
1. 监听指定owner的生命周期变化，在owner激活时，发送数据变更消息
2. 在数据变化且owner激活时，发送数据变更消息

#### 接口
* observe 注册监听器，使用指定的生命周期owner
* setValue 修改数据
* postValue 在主线程上修改数据

#### 回调
* onActive  只要有一个owner变成激活状态，就会被调用
* onInactive 所有的owner都变成未激活状态，才会被调用

### MutableLiveData类
* 包装LiveData，导出setValue, postValue接口

### MediatorLiveData

#### 功能
* 用于同时监控多个数据源（LiveData）

#### 接口
* addSource 添加一个数据源和对应的监听函数
* removeSource 移除数据源

#### 原理
* 将数据源和监听函数包装成一个Source对象，加入列表中
* 在onActive, onInactive回调中统一处理这些Source对象


### ComputableLiveData

#### 功能
* 用于后台在需要时生成数据

#### 原理
* 内部使用一个liveData，在liveData.onActive时，在后台线程运行一次刷新操作

#### 接口
* getLiveData 获取内部的liveData
* invalidate  设置数据无效，重新计算一次

#### 子类接口：
* compute  具体的计算过程


### Transformations 

#### map操作
* 用于将原值转成成另外一个值
* 使用MediatorLiveData实现这个功能


#### switchMap操作
* 用于将原值转成成另外一个LiveData
* 将Int值转成成一个列表



## support v4 

### SupportActivity类
* 实现了LifecycleOwner功能

### Fragment类
* 实现了LifecycleOwner功能