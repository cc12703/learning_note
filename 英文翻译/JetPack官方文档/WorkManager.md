

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

#### 灵活的运行间隔