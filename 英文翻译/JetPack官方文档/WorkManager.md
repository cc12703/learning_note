

# WorkManager

* 原始文档： https://developer.android.com/topic/libraries/architecture/workmanager

[TOC]




## 简介

### 概述
* WorkManager是一套API用于更容易的调度可延缓的、异步的任务，这些任务可以在应用退出或者设置重启后执行
* 这套API非常推荐来替换以前所有的后台调度APIs（FirebaseJobDispatcher, GcmNetworkManger, JobScheduler）
* WorkManager在一个现代的、一致的API中融合了前辈的特性，可以工作到API级别14，同时也更注意到电池寿命

#### 图示
![](https://gitee.com/cc12703/figurebed/raw/master/img/overview-criteria.png)


### 优点
#### 工作约束
* 使用工作约束可以来定义一些任务的最优条件
* 例子：只有Wi-Fi连接时，只有设备空闲时，只有设备存储空间充足时

#### 更稳定的调度
* 允许你调度任务只运行一次或重复运行，通过更灵活的调度窗口
* 任务可以被命名、打标记，允许你调度唯一、可替换的任务，监视或取消在一起的任务组
* 调度任务被存储在内部的SQLite数据库中，可以保证任务被持久化并在设备重启后进行重新调度
* WorkManager支持节能特征和最佳实践（像瞌睡模式）

#### 灵活的重试策略
* 当任务失败，WorkManager提供灵活的重试策略，包括一个指数退让策略

#### 任务链
* 对于复杂的相关任务，使用流畅的、原生的接口将独立任务链接在一起，可以允许你控制任务进行串行、并发运行
* 对于每个任务，你可以定义输入数据、输出数据
* 当任务链在一起时，WorkManager会自动在任务之间传递输出数据

```kotlin
WorkManager.getInstance(...)
    .beginWith(listOf(workA,workB))
    .then(workC)
    .enqueue()
```

#### 内建的线程互操作性
* WorkManger可以无缝的整合RxJava、协程、通过插件式的异步接口来提供灵活性


### 延时、可靠任务
* WorkManger适合以下特性的任务
    * 延时性：不要求马上运行
    * 可靠性：应用退出、设备重启后依然可以运行
* 例子：
    * 向后端服务器发送日志、分析数据
    * 周期性的和服务器同步数据
* WorkManager不并适合进程内的后台任务
    * 任务需要立刻执行
    * 应用退出后需要安全被结束




## 开始

### 依赖
* 需要在项目中导入库

```groovy
dependencies {
  def work_version = "2.4.0"

    // (只运行Java)
    implementation "androidx.work:work-runtime:$work_version"

    // Kotlin + 协程
    implementation "androidx.work:work-runtime-ktx:$work_version"

    // 可选 - RxJava2 support
    implementation "androidx.work:work-rxjava2:$work_version"

    // 可选 - GCMNetworkManager support
    implementation "androidx.work:work-gcm:$work_version"

    // 可选 - 测试辅助
    androidTestImplementation "androidx.work:work-testing:$work_version"
  }
```

### 定义任务
* 使用Worker类来定义任务
* doWork()函数会在WorkManger提供的后台线程上以同步方式运行
* doWork()中返回的Result，用于告知WorkManager服务器任务是否成功，如果失败任务是否要重试
    
#### Result结果
* Result.success() 任务成功结束
* Result.failure() 任务失败
* Result.retry()  任务失败，需要在另一个时刻进行重新运行

```kotlin
class UploadWorker(appContext: Context, workerParams: WorkerParameters):
       Worker(appContext, workerParams) {
   override fun doWork(): Result {

       // 进行工作，上传图像
       uploadImages()

       // 使用Result表明任务已经成功结束
       return Result.success()
   }
}
```

### 创建WorkRequest
* 任务一旦定义，就需要WorkManager服务按顺序进行调度
* WorkManager提供了很多灵活的方式来调度任务
    * 以一定时间间隔周期性运行任务
    * 只运行一次该任务
* 可以使用WorkRequest来选择如何调度任务
* WorkRequest定义了如何、何时来运行任务，最简单情况可以使用OneTimeWorkRequest

```kotlin
val uploadWorkRequest: WorkRequest =
   OneTimeWorkRequestBuilder<UploadWorker>()
       .build()
```

### 提交WorkRequest
* 最后提交WorkRequest给WorkManager，使用enqueue()
* 任务被执行的精确时间取决于WorkRequest中的约束、系统的优化

```kotlin
WorkManager
    .getInstance(myContext)
    .enqueue(uploadWorkRequest)
```




## 自定义WorkRequests
### 概述
* 为了让WorkManager可以调度任何任务，你需要创建一个WorkRequest对象并将其加入WorkManager的队列中
* WorkRequest包含了所有用于调度任务的信息
    * 约束信息
    * 调度信息：延时时间、间隔时间
    * 重试配置
    * 任务依赖的输入数据
* WorkRequest是一个抽象基类，由两个派生类可以用来创建请求
    * OneTimeWorkRequest：用于调度非周期性任务
    * PeriodicWorkRequest：用于调度按一定间隔运行的任务


```kotlin
val myWorkRequest = ...
WorkManager.getInstance(myContext).enqueue(myWorkRequest)
```

### 一次性调度
* 最简单情况，不需要附加配置，值使用一个静态方法
```kotlin
val myWorkRequest = OneTimeWorkRequest.from(MyWork::class.java)
```

* 复杂情况，需要使用一个构造器
```kotlin
val uploadWorkRequest: WorkRequest =
   OneTimeWorkRequestBuilder<MyWork>()
       // 额外配置
       .build()
```

### 周期性调度
#### 例子
* 周期性备份数据
* 周期性下载更新内容
* 周期性上传日志到服务器

#### 创建
```kotlin
val saveRequest =
       PeriodicWorkRequestBuilder<SaveImageToFileWorker>(1, TimeUnit.HOURS)
          // 额外配置
           .build()
```
说明
* 调度间隔：一个小时

#### 弹性运行间隔
* 如果任务对运行时间比较敏感，你可以配置PeriodicWorkRequest成在每个周期间隔运行一个弹性周期
* 为了定义弹性周期，需要跟随repeatInterval参数传入一个flexInterval参数
* 弹性周期开始于repeatInterval - flexInterval，结束于该周期的结束点
* repeatInterval必须要大于或等于PeriodicWorkRequest.MIN_PERIODIC_INTERVAL_MILLIS
* flexInterval必须要大于或等于 PeriodicWorkRequest.MIN_PERIODIC_FLEX_MILLIS

图示
![](https://gitee.com/cc12703/figurebed/raw/master/img/20210118224522.png)

示例
```kotlin
val myUploadWork = PeriodicWorkRequestBuilder<SaveImageToFileWorker>(
       1, TimeUnit.HOURS, // repeatInterval
       15, TimeUnit.MINUTES) // flexInterval
    .build()
```

#### 周期性任务的约束作用
* 可以对周期性任务设置约束条件，例子：可以设置设备在充电时才运行
* 这种情况下就算传入了repeatInterval, PeriodicWorkRequest也不会运行除非条件达到
* 这将导致任务被延后运行，甚至被忽略如果在运行周期内条件没有满足的话



### 约束条件
#### 概述
* 约束条件会确保任务被延后直到满足最优条件
* 当指定多个条件时，只有当所有条件都满足时任务才会运行
* 在任务运行时若有条件没有满足，则WorkManager会停止对应任务的运行
* 等到所有条件都满足后，任务会被重试运行

#### 有效约束
* NetworkType：约束任务可以运行的网络类型
* BatteryNotLow：若为True，任务不会在设备低功耗模式下运行
* RequiresCharging：若为True，任务只会在设备充电时运行
* DeviceIdle：若为True，任务只会在设备空闲时才会运行
* StorageNotLow：若为True，任务不会在设备存储空间太低时运行

#### 步骤
1. 使用Contraints.Builder()创建一个约束实例
1. 将其赋值给WorkRequest.Builder()

```kotlin
val constraints = Constraints.Builder()
   .setRequiredNetworkType(NetworkType.UNMETERED)
   .setRequiresCharging(true)
   .build()

val myWorkRequest: WorkRequest =
   OneTimeWorkRequestBuilder<MyWork>()
       .setConstraints(constraints)
       .build()
```

### 延时任务
* 正常情况下如果任务没有设置条件，或者在入队列时所有条件都满足，系统会立刻选择一个任务运行
* 如果你不想任务马上被运行，可以设置一个延时时间，在该时间后任务才会运行

```kotlin
val myWorkRequest = OneTimeWorkRequestBuilder<MyWork>()
   .setInitialDelay(10, TimeUnit.MINUTES)
   .build()
```


### 重试和回避策略
#### 概述
* 如果想让WorkManager重试运行任务，可以从任务中返回Result.retry()
* 任务会根据回退延时时间和回退策略进行重新调度

#### 定义
* 回退延时时间：在首次尝试尝试后，重试运行任务前的最小等待时间
* 回退策略：定义了回退延时时间如何随着重试次数的增加而增加
    * WorkManager有两种策略： LINEAR and EXPONENTIAL
* 每个工作任务队列都有一个回退延时时间、回退策略
    * 默认策略是 EXPONENTIAL
    * 默认延时时间是 10秒

#### 示例
```kotin
val myWorkRequest = OneTimeWorkRequestBuilder<MyWork>()
   .setBackoffCriteria(
       BackoffPolicy.LINEAR,
       OneTimeWorkRequest.MIN_BACKOFF_MILLIS,
       TimeUnit.MILLISECONDS)
   .build()
```
说明
* 例子中最小回退延时时间设置成了最小的允许时间，10秒
* 回退策略是LINEAR，重试间隔会大概每次新尝试后增加10秒
* 举例：如果首次运行结束后返回Result.retry()，将会在10秒后重新运行一次
    * 如果接下来都返回Result.retry()，间隔会变成20秒，30秒，40秒
* 如果回退策略被设置成EXPONENTIAL，重试间隔会接近于20秒，40秒，80秒


### 标记任务
* 每个任务都会有一个唯一标识，用于取消任务和监控任务进度
* 如果你有一组逻辑上相关的任务，你会发现给任务打上标记会非常有用
* 标记允许你对一组任务进行操作
* 一个任务请求可以打上多个标记，在内部标记被存储为一个字符串集合

#### 举例
* WorkManager.cancelAllWorkByTag(String) 取消所有有特定标记的任务
* WorkManager.getWorkInfosByTag(String) 返回一个WorkInfo对象的列表，用于确定当前的任务状态

```kotlin
val myWorkRequest = OneTimeWorkRequestBuilder<MyWork>()
   .addTag("cleanup")
   .build()
```

### 指定输入数据
* 任务有时候需要输入数据以便进行工作
    * 例子：任务上传图片，需要图标的URI作为输入
* 输入值被以KV键的形式存储在Data对象中，再被设置给请求对象
* WorkManger会在执行任务时，将输入数据传递给任务
* Worker类可以通过调用Worker.getInputData()来获取输入值

#### 示例
```kotlin
// 定义带输入值的Worker
class UploadWork(appContext: Context, workerParams: WorkerParameters)
                    : Worker(appContext, workerParams) {

   override fun doWork(): Result {
       val imageUriInput =
           inputData.getString("IMAGE_URI") ?: return Result.failure()

       uploadFile(imageUriInput)
       return Result.success()
   }
   ...
}

// 创建一个WorkRequest对象，带输入值
val myUploadWork = OneTimeWorkRequestBuilder<UploadWork>()
   .setInputData(workDataOf(
       "IMAGE_URI" to "http://..."
   ))
   .build()
```




## 任务状态

* 任务在其生命周期中会经历一系列的State变更

### 一次性任务状态
* 对于一次性任务请求，任务开始于 ENQUEUED 状态
* 在ENQUEUED状态，当前所有约束条件满足、延时时间到达，任务会进行RUNNING状态
* 根据任务的输出，状态会变成 SUCCEEDED、FAILED、重新回到ENQUEUED状态
* 在过程中的任何点上，如果任务被取消则进入CANCELLED状态
* SUCCEEDED、FAILED、CANCELLED代表了终端状态，在这些状态下WorkInfo.State.isFinished()会返回true

图示
![](https://gitee.com/cc12703/figurebed/raw/master/img/20210119102334.png)


### 周期任务状态
* 对于周期任务，只有一个终端状态CANCELLED
* 周期任务没有结束，在每次运行后都会被重新调度，无论什么结构

图示
![](https://gitee.com/cc12703/figurebed/raw/master/img/20210119102717.png)


### 阻塞状态
* BLOCKED状态是一个不会被注意到的最终状态
* 该状态用于任务链的情况




## 管理任务

### 概述
* 一旦你定义了任务和任务请求，最后一步就是将任务加入队列
* 最简单的方法就是调用WorkManager的enqueue()方法

#### 风险
* 在把任务加入队列时需要小心，避免重复加入队列
* 例子：应用想每24小时上传日志到后端服务器，如果不小心最终你可能会将同一任务加入队列很多次
* 为了保证任务只加入一次，你可以将任务作为**唯一任务**来调度

### 唯一任务
* 唯一任务是一个很强大的概念，可以保证带特定名字的任务一次只有一个实例
* 不像IDs，唯一名字是具有可读性的，由开发者定义，会替换掉WorkManger自动生成的名字
* 不像标记，唯一名字只能分配给一个任务实例

#### 使用
* 唯一任务可以用于一次性任务和周期性任务
* 使用接口
    * WorkManager.enqueueUniqueWork() 用于一次性任务
    * WorkManager.enqueueUniquePeriodicWork() 用于周期性任务
* 接收的参数
    * uniqueWorkName 一个字符串用于唯一标识任务请求
    * existingWorkPolicy 一个枚举用于告知WorkManager，如果已经存在同名任务该如何处理
    * work 要调度的WorkRequest对象

#### 示例
```kotlin
val sendLogsWorkRequest =
       PeriodicWorkRequestBuilder<SendLogsWorker>(24, TimeUnit.HOURS)
           .setConstraints(Constraints.Builder()
               .setRequiresCharging(true)
               .build()
            )
           .build()
WorkManager.getInstance(this).enqueueUniquePeriodicWork(
           "sendLogs",
           ExistingPeriodicWorkPolicy.KEEP,
           sendLogsWorkRequest
)
```
说明
* 如果WorkManager发现已经有sendLogs任务在队列中了，已存在的任务会被保留，新的任务将不会被加入

#### 用途
* 如果你要慢慢构建一个长任务时，唯一任务序列会非常有用
* 例子：一个图片编辑应用，让用户恢复一个长的动作链
    * 每个恢复操作都需要执行一段时间
    * 这些操作都需要按正确的顺序来执行
    * 这时候应用可以创建一个名为undo的链，在需要时将每个恢复操作加入链中


#### 冲突处理策略
* 当使用唯一任务时，需要告知WorkManager当出现冲突时要如何处理
* 可以在任务加入队列时，传入一个枚举值

##### 一次性任务
* 枚举类为 ExistingWorkPolicy，一共4个选项
* REPLACE：取消已存在的任务，使用新任务替换掉
* KEEP：保留已存在的任务，忽略新任务
* APPEND：新任务会被加在已存在任务的后面，导致新任务被链接上已存在任务
        * 已存在任务会变成新任务的前提条件，如果已存在任务变成CANCELLED或FAILED状态，新任务也会变成该状态
* APPEND_OR_REPALCE：非常像APPEND，除了已存在任务**不会变成**新任务的前提条件

##### 周期性任务
* 枚举类为 ExistingPeriodicWorkPolicy，一个2个选项
* REPLACE：同ExistingWorkPolicy
* KEEP：同ExistingWorkPolicy


### 监测任务
#### 概述
* 将任务加入队列后，就可以使用名字、ID、标记，通过WorkManager来查询任务的状态
* 查询返回一个ListenableFuture形式的WorkInfo对象
    * 任务ID
    * 标记
    * 当前状态
    * 输出数据，通过Result.success(outputData)返回的
* 每个方法的LiveData变体，可以允许你通过注册一个listener来监测WorkInfo对象的变更

查询
```kotlin
// by id
workManager.getWorkInfoById(syncWorker.id) // ListenableFuture<WorkInfo>

// by name
workManager.getWorkInfosForUniqueWork("sync") // ListenableFuture<List<WorkInfo>>

// by tag
workManager.getWorkInfosByTag("syncTag") // ListenableFuture<List<WorkInfo>>
```

监测
```kotlin
workManager.getWorkInfoByIdLiveData(syncWorker.id)
               .observe(viewLifecycleOwner) { workInfo ->
   if(workInfo?.state == WorkInfo.State.SUCCEEDED) {
       Snackbar.make(requireView(), 
      R.string.work_completed, Snackbar.LENGTH_SHORT)
           .show()
   }
}
```

#### 复杂查询
* WorkManager 2.4.0版本支持使用WorkQuery来进行复杂的查询
* WorkQuery通过组合标记、任务状态、唯一任务名字来进行查询

```kotlin
val workQuery = WorkQuery.Builder
       .fromTags(listOf("syncTag"))
       .addStates(listOf(WorkInfo.State.FAILED, WorkInfo.State.CANCELLED))
       .addUniqueWorkNames(listOf("preProcess", "sync")
    )
   .build()

val workInfos: ListenableFuture<List<WorkInfo>> 
            = workManager.getWorkInfos(workQuery)
```
说明
* 每个组件（标记、状态、名字）都是**与**在一起的
* 每个组件的值都是**或**在一起的
* 例子：<em>(name1 OR name2 OR ...) AND (tag1 OR tag2 OR ...) AND (state1 OR state2 OR ...)</em>


### 取消和停止任务
* 如果不在需要以前加入队列的任务运行，你可以将任务取消掉
* WorkManager会检查任务状态
    * 如果任务已经运行完成，则不会发生任何事情
    * 否则任务状态会变成CANCELLED，在未来不会在运行。任何依赖该任务的请求也会变成CANCELLED
* 当前正在运行的任务，会接收到一个方法调用 ListenableWorker.onStopped()，可以重载该函数进行资源清理

```kotlin
// by id
workManager.cancelWorkById(syncWorker.id)

// by name
workManager.cancelUniqueWork("sync")

// by tag
workManager.cancelAllWorkByTag("syncTag")
```


### 停止正在运行的任务
#### 被WorkManager停止的原因
* 明确要求任务取消（通过调用WorkManager.cancelWorkById(UUID)）
* 对于唯一任务，明确用REPLACE策略加入新任务请求，旧的任务请求会立刻被取消
* 任务约束不在满足
* 系统告知应用停止任务

#### 资源释放
* 在任务被中止时，需要配置释放掉占用的资源
* 有两种方法可以让我们知道任务被中止了
    * onStopped()回调：WorkManager会触发ListenableWorker.onStopped()方法
    * isStopped()函数：调用ListenableWorker.isStopped()检查任务是否被停止了




## 观测进度

### 概述
* WorkManager2.3.0支持设置和观测任务的中间进度
* 如果任务运行时应用在前台，可以通过LiveData形式的WorkInfo将信息展现给用户

#### ListenableWorker
* 提供setProgressAsyn()接口，允许开发者设置中间进度，又界面进行观察
* 进度信息是Data类型的，是一个可串行化属性的容器
* 当ListenableWorker运行时，进度信息可以被观察和更新。当worker执行完成后在设置，信息会被忽略
* 通过getWorkInfoBy...()和getWorkInfoBy...LiveData()来观察进度信息
    * 该方法返回一个WorkInfo实例，带getProgress()方法返回Data


### 更新进度
* Kotlin而言，可以使用CoroutinWorker的setProgress()来更新进度

```kotlin
import android.content.Context
import androidx.work.CoroutineWorker
import androidx.work.Data
import androidx.work.WorkerParameters
import kotlinx.coroutines.delay

class ProgressWorker(context: Context, parameters: WorkerParameters) :
    CoroutineWorker(context, parameters) {

    companion object {
        const val Progress = "Progress"
        private const val delayDuration = 1L
    }

    override suspend fun doWork(): Result {
        val firstUpdate = workDataOf(Progress to 0)
        val lastUpdate = workDataOf(Progress to 100)
        setProgress(firstUpdate)
        delay(delayDuration)
        setProgress(lastUpdate)
        return Result.success()
    }
}
```


### 观察进度
* 使用getWorkInfoBy...()和getWorkInfoBy...LiveData()

```kotlin
WorkManager.getInstance(applicationContext)
    // requestId 是 任务请求的ID号
    .getWorkInfoByIdLiveData(requestId)
    .observe(observer, Observer { workInfo: WorkInfo? ->
            if (workInfo != null) {
                val progress = workInfo.progress
                val value = progress.getInt(Progress, 0)
                ...
            }
    })
```





## 链接任务

### 概述
* WorkManager允许创建一个任务链，并加入运行队列
    * 任务链包含多个相互依赖的任务，需要按一定顺序运行
* 该功能适合于需要按特定顺序运行很多任务


### 操作
#### 创建
* 使用WorkManager.beginWith(OneTimeWorkRequest)
* 使用WorkManager.beginWith(List<OneTimeWorkRequest>)
* 返回一个WorkContinuation对象

#### 添加
* 使用WorkContinuation.then(OneTimeWorkRequest)
* 使用WorkContinuation.then(List<OneTimeWorkRequest>)
* 返回一个新的WorkContinuation对象
* 如果添加一个OneTimeWorkRequest实例的列表，这种请求会并行运行

#### 加入队列
* 使用 WorkContinuation.enqueue()

```kotlin
WorkManager.getInstance(myContext)
   // 候选者并行运行
   .beginWith(listOf(plantName1, plantName2, plantName3))
   // 依赖任务（只在前面任务运行完成后才能执行）
   .then(cache)
   .then(upload)
   // 调用enquque，开始运行
   .enqueue()
```


### 输入合并
* 当链接OneTimeWorkRequest实例时，父任务的输出会传递给子任务作为输入
* 上个例子中，plantName1,plantName2, plantName3的输出会传递给cahce
* 为了管理从多个父任务来的输入，WorkManager使用InputMerger
* 有两个自类型
    * OverwritingInputMerger：将所有输入的键添加到输出，如果键有冲突，则使用最新加入的值
    * ArrayCreatingInputMerger：合并输入，必要是创建数组

#### OverwritingInputMerger
* 是默认使用的合并方法
* 如果合并是有键冲突，则使用该键最新的值覆盖前面的值


##### 无冲突
![](https://gitee.com/cc12703/figurebed/raw/master/img/20210120101958.png)

##### 有冲突
![](https://gitee.com/cc12703/figurebed/raw/master/img/20210120102025.png)

说明
* 因为请求是并发运行，所有无法保证各个任务的运行顺序
* plantName1的值 tulip 和 elm 都有可能


#### ArrayCreatingInputMerger
* 可能保留所有任务的输出
* 输出形式为 键<-->列表

```kotlin
val cache: OneTimeWorkRequest = OneTimeWorkRequestBuilder<PlantWorker>()
   .setInputMerger(ArrayCreatingInputMerger::class)
   .setConstraints(constraints)
   .build()
```

##### 无冲突
![](https://gitee.com/cc12703/figurebed/raw/master/img/20210120102812.png)

##### 有冲突
![](https://gitee.com/cc12703/figurebed/raw/master/img/20210120102837.png)


### 任务状态
* OneTimeWorkRequest链会顺序执行，只要前面的任务运行成功
* 如果运行中任务失败了或被取消了，会对依赖的任务请求产生下游影响

#### 开始执行
* 当任务链的第一个OneTimeWorkRequest进入运行队列时
* 所有的子任务请求会进入BLOCKED状态，知道第一个任务请求完成
* 一旦任务约束满足，第一个任务请求就会运行

图示
![](https://gitee.com/cc12703/figurebed/raw/master/img/20210120105529.png)


#### 首任务执行成功
* 如果首个OneTimeWorkRequest 或者 List<OneTimeWorkRequest> 运行成功
* 下一级依赖的任务请求就会被加入运行队列
* 只要每个任务都运行成功，相同的模式就会在链的剩余部分进行传播

图示
![](https://gitee.com/cc12703/figurebed/raw/master/img/20210120110009.png)


#### 错误重试
* 当任务运行时发生了一个错误后，任务会根据定义的回退策略进行重试
* 重试请求也是任务链的一部分：请求会使用提供给它的输入数据进行重试，并行运行的任务不受影响

图示
![](https://gitee.com/cc12703/figurebed/raw/master/img/20210120111434.png)


#### 错误失败
* 如果重试策略为设置，任务直接返回Result.failure()。则任务请求和所有依赖请求都会被标记为FAILED

图示
![](https://gitee.com/cc12703/figurebed/raw/master/img/20210120111707.png)


#### 任务取消
* 当一个OneTimeWorkRequest被取消后，会发生相同的逻辑。所有依赖的请求都会标记为CANCELLED
* 如果对有失败、取消请求的链增加更多的请求时，新加入的请求也会被标记成FAILED或CANCELLED

图示
![](https://gitee.com/cc12703/figurebed/raw/master/img/20210120111855.png)


#### 建议
* 当创建任务请求链时，相互依赖的请求最好要定义重试策略，以确保任务能及时的完成
    * 失败的任务请求会导致出现未完成的请求链




## 自定义配置

### 概述
* 默认情况下，WorkManager会在应用启动时自动进行配置，使用对大部分应用合理的配置选项
* 如果你想对WormManager管理、调度任务有更多的控制，可以自定义配置
* 自定义初始化用的最多的方式就是 按需初始化 模式，该模式在WorkManger2.1.0提供


### 按需初始化
* 按需初始化可以让你在需要该组件时才创建WorkManager，而不是在应用启动时就创建
* 这样做可以将WorkManager从启动流程中移除，提升应用启动速度

#### 移除默认初始化器
* 首先移除默认的初始化器
* 使用tools:node="remove" 更新AndroidManifest.xml

```xml
<provider
    android:name="androidx.work.impl.WorkManagerInitializer"
    android:authorities="${applicationId}.workmanager-init"
    tools:node="remove" />
```

#### 实现Configuratino.Provider
* 让Application类实现Configuratino.Provider接口
* 通过Configuration.Provider.getWorkManagerConfiguration()提供自定义配置
* 使用WorkManager.getInstance(Context)来获取WorkManager
    * WorkManger会使用自定义的getWorkManagerConfiguration()来发现配置


```kotlin
class MyApplication() : Application(), Configuration.Provider {
     override fun getWorkManagerConfiguration() =
           Configuration.Builder()
                .setMinimumLoggingLevel(android.util.Log.INFO)
                .build()
}
```



## 线程和并发

### 概述
* 在开头时我们提到过，WorkManager会异步的执行后台任务
* 基本实现只满足大部分应用的需求，对于更多高级的用例(像正确处理任务停止)，需要你学习线程和并发

#### 任务类型
##### Worker
* 是最简单的一种实现
* WorkManager会自动在一个后台线程中运行任务

##### CoroutineWorker
* 推荐Kotlin用户使用
* CoroutineWorker实例会为后台任务暴露一个挂起函数
* 默认情况下任务由一个默认的Dispatcher执行，你也可以自定义该Dispatcher

##### RxWorker
* 推荐RxJava用户使用
* 如果你的异步代码是使用RxJava建立的，则推荐使用
* 在RxJava中你可以自由的选择线程策略

##### ListenableWorker
* 是Worker,CoroutineWorker,RxWorker的一个基础类
* 是给哪些需要和基于回调的异步接口进行交互的Java开发者准备的


### Worker任务
* 当使用Worker时，WorkManager会在后台线程中自动调用Worker.doWork()
* 后台线程来自于Executor线程池，在WorkManger的Configuration中被指定
* 默认情况下，WorkManager会设置一个Executor。但是你也可以自定义一个。

```kotlin
WorkManager.initialize(
    context,
    Configuration.Builder()
         // 使用只有8个线程的固定线程池
        .setExecutor(Executors.newFixedThreadPool(8))
        .build())
```

#### 实现
* 注意Worker.doWork()是一个同步调用
    * 在后台任务中做的所有操作都必须为阻塞模式
    * 如果在doWork()中调用了一个异步API，并且返回了一个Result。你的回调不会被正确处理


```kotlin
class DownloadWorker(context: Context, params: WorkerParameters) 
                : Worker(context, params) {

    override fun doWork(): ListenableWorker.Result {
        repeat(100) {
            if (isStopped) {
                break
            }

            try {
                downloadSynchronously("https://www.google.com")
            } catch (e: IOException) {
                return ListenableWorker.Result.failure()
            }

        }
        return ListenableWorker.Result.success()
    }
}
```

### CoroutineWorker任务
* 对于Kotline用户，WorkManager提供了对协程的支持
* 需要在gradle文件中包含work-runtime-ktx库
* CoroutineWorker会自动处理停止操作（取消协程运行，传递取消信号）


#### 实现
* CoroutineWorker的doWork()是一个suspend函数，并不在Executor中运行
* 默认在Dispatchers.Default中运行，也可以自定义提供自己的CoroutineContext

默认分发器
```kotlin
class CoroutineDownloadWorker(
    context: Context,
    params: WorkerParameters
) : CoroutineWorker(context, params) {

    override suspend fun doWork(): Result = {
        val data = downloadSynchronously("https://www.google.com")
        saveData(data)

        Result.success()
    }
}
```

自定义分发器
```kotlin
class CoroutineDownloadWorker(
    context: Context,
    params: WorkerParameters
) : CoroutineWorker(context, params) {

    override suspend fun doWork(): Result {
        withContext(Dispatchers.IO) {
            val data = downloadSynchronously("https://www.google.com")
            saveData(data)
            return Result.success()
        }

    }
}
```



### ListenableWorker任务
* 在特定情况下，你可能需要提供一个自定义的线程策略
    * 例如：需要处理基于回调的异步操作
* 这个时候你就不能简单的使用Worker了，因为该类在阻塞模式下无法处理这种情况
* WorkManager提供了ListenableWorker来处理这种情况

#### 说明
* ListenableWorker是一个基础API类，Worker,CoroutineWorker,RxWorker都派生自该类
* 当任务开始、结束时，ListenableWorker只会发送一个信号，什么时候离开线程取决于你自己
* 任务开始信号在主线程被发送，所以你手动进入一个后台线程就非常重要
*  ListenableWorker.startWork() 会返回一个ListenableFuture形式的Result实例


#### 实现
* 创建ListenableFuture方式
    * 使用Guava库中的ListeningExecutorService
    * 使用councurrent-futures库中的CallbackToFutureAdapter

```kotlin
class CallbackWorker(
        context: Context,
        params: WorkerParameters
) : ListenableWorker(context, params) {
    override fun startWork(): ListenableFuture<Result> {
        return CallbackToFutureAdapter.getFuture { completer ->
            val callback = object : Callback {
                var successes = 0

                override fun onFailure(call: Call, e: IOException) {
                    completer.setException(e)
                }

                override fun onResponse(call: Call, response: Response) {
                    ++successes
                    if (successes == 100) {
                        completer.set(Result.success())
                    }
                }
            }//end of Callback

            completer.addCancellationListener(cancelDownloadsRunnable, executor)

            repeat(100) {
                downloadAsynchronously("https://example.com", callback)
            }

            callback
        }
    }//end of startWork
}
```




## 长期运行的任务

### 概述
* WorkManager2.3.0-alpha12开始支持长期运行的任务
* WorkManager可以给OS一个信号，当任务执行时进程可以尽可能的存活着
* 这些任务可以运行超过10分钟
* 实现层面WorkManager会已应用的名义运行一个前台服务，来执行这些任务请求。会显示一个可配置的通知栏

#### 接口
* ListenableWorker提供了setForegroundAsync接口
* CoroutineWorker提供了setForeground接口
* 这些接口允许开发者指定某些任务请求非常重要，或者需要长时间运行

#### 取消
* WorkManager允许你创建一个PendingIntent，用于在无需注册一个android组件时取消掉任务


### 创建和管理
#### Java
* 开发者可以使用ListenableWorker或者Worker，调用setForegroundAsync接口
* 该接口返回一个ListenableFuture<Void>
* 你也可以在调用该接口时上传一个持续显示的通知栏

```kotlin
public class DownloadWorker extends Worker {
    private static final String KEY_INPUT_URL = "KEY_INPUT_URL";
    private static final String KEY_OUTPUT_FILE_NAME = "KEY_OUTPUT_FILE_NAME";

    private NotificationManager notificationManager;

    public DownloadWorker(
        @NonNull Context context,
        @NonNull WorkerParameters parameters) {
            super(context, parameters);
            notificationManager = (NotificationManager)
                context.getSystemService(NOTIFICATION_SERVICE);
    }

    @NonNull
    @Override
    public Result doWork() {
        Data inputData = getInputData();
        String inputUrl = inputData.getString(KEY_INPUT_URL);
        String outputFile = inputData.getString(KEY_OUTPUT_FILE_NAME);
        // 标记Worker非常重要
        String progress = "Starting Download";
        setForegroundAsync(createForegroundInfo(progress));
        download(inputUrl, outputFile);
        return Result.success();
    }

    private void download(String inputUrl, String outputFile) {
       // 下载一个文件、更新读取的字节数
       // 当需要更新通知栏时，周期性的调用setForegroundInfoAsync()
    }

    @NonNull
    private ForegroundInfo createForegroundInfo(@NonNull String progress) {
        // 使用 已读字节数、内容长度生成一个通知栏

        Context context = getApplicationContext();
        String id = context.getString(R.string.notification_channel_id);
        String title = context.getString(R.string.notification_title);
        String cancel = context.getString(R.string.cancel_download);
        // PendingIntent 用于取消该任务
        PendingIntent intent = WorkManager.getInstance(context)
                .createCancelPendingIntent(getId());

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            createChannel();
        }

        Notification notification = new NotificationCompat.Builder(context, id)
                .setContentTitle(title)
                .setTicker(title)
                .setSmallIcon(R.drawable.ic_work_notification)
                .setOngoing(true)
                // 给通知栏增加一个取消动作
                .addAction(android.R.drawable.ic_delete, cancel, intent)
                .build();

        return new ForegroundInfo(notification);
    }

    @RequiresApi(Build.VERSION_CODES.O)
    private void createChannel() {
        // 创建一个通知栏通道
    }
}
```

#### Kotlin
* 开发者使用CoroutineWorker，对应的setForegourd()接口

```kotlin
class DownloadWorker(context: Context, parameters: WorkerParameters) :
    CoroutineWorker(context, parameters) {

    private val notificationManager =
        context.getSystemService(Context.NOTIFICATION_SERVICE) as
                NotificationManager

    override suspend fun doWork(): Result {
        val inputUrl = inputData.getString(KEY_INPUT_URL)
                       ?: return Result.failure()
        val outputFile = inputData.getString(KEY_OUTPUT_FILE_NAME)
                       ?: return Result.failure()
         // 标记Worker非常重要
        val progress = "Starting Download"
        setForeground(createForegroundInfo(progress))
        download(inputUrl, outputFile)
        return Result.success()
    }

    private fun download(inputUrl: String, outputFile: String) {
        // 下载一个文件、更新读取的字节数
       // 当需要更新通知栏时，周期性的调用setForegroundInfo()
    }


    private fun createForegroundInfo(progress: String): ForegroundInfo {
        val id = applicationContext.getString(R.string.notification_channel_id)
        val title = applicationContext.getString(R.string.notification_title)
        val cancel = applicationContext.getString(R.string.cancel_download)
        // PendingIntent 用于取消该任务
        val intent = WorkManager.getInstance(applicationContext)
                .createCancelPendingIntent(getId())

        // Create a Notification channel if necessary
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            createChannel()
        }

        val notification = NotificationCompat.Builder(applicationContext, id)
            .setContentTitle(title)
            .setTicker(title)
            .setContentText(progress)
            .setSmallIcon(R.drawable.ic_work_notification)
            .setOngoing(true)
             // 给通知栏增加一个取消动作
            .addAction(android.R.drawable.ic_delete, cancel, intent)
            .build()

        return ForegroundInfo(notification)
    }

    @RequiresApi(Build.VERSION_CODES.O)
    private fun createChannel() {
        // Create a Notification channel
        // 创建一个通知栏通道
    }

    companion object {
        const val KEY_INPUT_URL = "KEY_INPUT_URL"
        const val KEY_OUTPUT_FILE_NAME = "KEY_OUTPUT_FILE_NAME"
    }
}
```