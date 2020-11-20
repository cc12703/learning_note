


# RxJava编程


## 特点
* 异步操作库
* 让异步操作简洁（随着程序逻辑越来越复杂，依然能够保持简洁）
* 将复杂逻辑穿成一条调用链

## 原理
* 使用通用形式的观察者模式来实现异常操作

### 概念
* Observable  可观察者，即被观察者
* Observer  观察者
* subscribe  订阅
* event 事件

### 回调
* onNext() 普通事件
* onCompleted()  通知观察者Observable没有更多的数据
* onError()  事件队列异常，即观察者有错误出现

### 操作

#### 创建Observer
* 实现Observer接口
* 继承Subscriber类

#### 创建Observable
* 使用create()方法
* 使用just(T...)方法，将传入的参数依次发送出来
* 使用from(T[])方法，将传入的数组拆分成具体对象后，依次发送出来

#### 订阅
* 使用subscribe()方法,例子 Observable.subscribe(Subscriber)


## 线程控制

### Scheduler 
* 调度器，线程控制器
* 指定每一段代码应该运行在什么样的线程

### 类型
* Schedulers.immediate()  直接在当前线程运行
* Schedulers.newThread()  总是启用新线程
* Schedulers.io()   I/O 操作，内部使用无上限线程池
* Schedulers.computation()  用于CPU密集计算，内部使用固定线程池

### 操作
* subscribeOn()： 指定事件产生的线程
* observeOn():  指定事件消费的线程


## 变换

### 作用
* 将事件序列中的对象或整个序列进行加工处理，转换成不同的事件或事件序列

### map
* 进行一对一对象的变换

### flatmap
* 进行一对多对象的变换

#### 步骤
1. 使用传入的事件创建一个Observable 对象
2. 激活该Observable 对象，使其发送事件
3. 这些事件都被汇入同一个 Observable
4. 这个 Observable 负责将这些事件统一交给 Subscriber


### 实现原理
* 通过lift操作，即一种代理机制
* 通过事件拦截和处理实现事件序列的变换
* 每次执行lift后都会生成一个新的Observable，
* 该Observable会接收原始Observable发出的事件，并在处理后发送给Subscriber

### compose
* compose(Transformer)： 对 Observable 自身进行变换


### 线程控制
* subscribeOn() 和 observeOn() 的内部实现，也是利用的 lift()操作。
* subscribeOn()在OnSubscribe中切换了线程，即在它通知上一级 OnSubscribe 时。
* observeOn() 在内建的 Subscriber 中切换了线程。


## 应用场景

* 网络: RxJava + Retrofit
* 界面回调: RxBinding
* 消息


## 背压(Backpressur)

### 定义
* 在异步场景中，被观察者发送事件速度远快于观察者的处理速度的情况下
* 一种告诉上游的被观察者降低发送速度的策略
* **背压是流速控制的一种策略**

### 策略

#### 响应式拉取
* 观察者主动从被观察者那里去拉取数据，
* 而被观察者变成被动的等待通知再发送数据。


### 其他流控方式

#### 过滤（抛弃）
* 虽然生产者产生事件的速度很快，但是把大部分的事件都直接过滤（浪费）掉。

#### 缓存
* 虽然生产者产生事件的速度很快，但是可以选择先缓存一部分，然后慢慢读。

#### 操作符
##### onBackpressurebuffer
* 把observable发送出来的事件做缓存
* 当request方法被调用的时候，给下层流发送缓存的事件

##### onBackpressureDrop
* 把observable发送的事件抛弃掉
* 当request方法被调用的时候，给下层流发送之后的事件


## 2.0版本

### 观察者模式
* Observeable用于订阅Observer，是不支持背压的
* Flowable用于订阅Subscriber，是支持背压的


### 创建observable

#### just()
* 用于将多个对象转换成observable

```java
	Observable<String> observable 
            = Observable.just("foo", "bar");
```

#### from()
* 用于将数组，列表和回调函数转换成observable

```java
Observable<String> observable 
        = Observable.fromIterable(Arrays.asList("foo", "bar"));

Observable<String> observable 
        = Observable.fromArray("foo", "bar");

Observable<String> observable 
        = Observable.fromCallable(() -> {
                Thread.sleep(1000);
                return "foo";
            });
```

#### create()
* 从函数中创建一个observable

```java
Observable<String> observable = Observable.create(new ObservableOnSubscribe<String>() {
        @Override
        public void subscribe(ObservableEmitter<String> observableEmitter) throws Exception {
            observableEmitter.onNext("foo");
            observableEmitter.onNext("bar");
            observableEmitter.onComplete();
        }
    });
```

#### defer()
* 用于给每个监听者创建一个新的observable
* 延迟到监听时刻才会创建

```java
Observable<String> observable = Observable.defer(() -> Observable.just("foo", "bar"));
```


#### range()
* 创建一个observable，用于发送一串连续的数字

```java
Observable<Integer> observable = Observable.range(0, 3);
```


#### timer()
* 创建一个定时发送对象的observable

