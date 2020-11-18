

# OKHttp学习


## 特点

* 支持SPDY，HTTP2
* 支持连接池
* 支持GZIP压缩
* 支持缓存响应
* 使用Interceptors来处理请求和响应


## 关键类

* OkHttpClient  对外接口类
* Request 请求数据类
* Request.Builder  请求生成器
* Dispatcher 请求分发器（异步，同步执行请求）
* Interceptor 拦截器接口类
* Route 路由信息
* Address  地址信息


## 模块

### 接口
* Call
* OkHttpClient
* Dispatcher
* Request

### 协议
* HTTP
* TLS
* WebSocket

### 连接
* Route
* Dns
* RealConection
* StreamAllocation

### 缓存
* DiskLRUCache
* CacheStrategy

### IO
* Http1Codec
* Http2Codec
* okio


## 运行机制

### 分发器
以下几个类组成

#### Call
* 请求调用器接口类
* 支持同步调用，异步调用

#### RealCall
* 请求调用类，实现了Call接口

#### Dispatcher
* 请求分发器，调度异步运行的请求
* 使用线程池来运行异步请求



### 拦截器
以下几个类组成

#### Interceptor
* 拦截器接口类
* 所有拦截器都需要实现 Response intercept(Chain chain) 接口
* 从拦截器链中生成响应

#### Interceptor.Chain
* 拦截器链接口类
* 对应的实现类为RealInterceptorChain
* 拦截器可以使用该接口，进入到下一级处理

#### RealCall.getResponseWithInterceptorChain
* 启动拦截器链
* 逻辑：
    1. 构建一个拦截器堆栈
    2. 生成RealInterceptorChain对象
    3. 调用链的proceed接口，开始处理请求

#### RealInterceptorChain.proceed
* 逻辑：
    1. 检查拦截器是否已遍历完成
    2. 检查运行状态是否正常
    3. 从堆栈中获取下一个拦截器
    4. 调用其intercept接口


#### 拦截器堆栈
1. 用户自定义的全局拦截器
2. RetryAndFollowUpInterceptor   处理错误和重定向
3. BridgeInterceptor   用于桥接应用层和网络层
4. CacheInterceptor  处理缓存
5. ConnectInterceptor  生成新连接
6. 用户自定义的网络拦截器
7. CallServerInterceptor  访问远端服务器


## 缓存机制


### 概述
* 将请求url进行md5，用于缓存文件定位
* 定时进行缓存清理
* 使用操作日志来保证缓存的完整性

### 模块
* CacheInterceptor 实现主逻辑
* Cache                  实现缓存功能 
* DiskLruCache    实现缓存文件的操作
* FileSystem       包装文件系统
* CacheStrategy  实现缓存策略


### 数据保存
* 每个url对应一组文件，使用url的MD5值作为文件名
* 缓存文件有两个类型：元数据（meta-data）,  响应数据（body）
* 元数据是 Cache.Entry对象的序列化

### 缓存清理
* 逻辑：DiskLruCache.cleanupRunnable
    1. 判断是否可以运行
    2. 缩减缓存文件所占的空间 trimToSize()
    3. 缩减日志文件所占的空间 rebuildJournal()


### 操作日志
* 名字：journal
* 格式：文本，按行分隔
* 内容：头信息，操作记录

#### 头信息
* 识别符 libcore.io.DiskLruCache
* 版本号
* 应用版本号
* 操作记录个数

#### 操作记录

##### CLEAN操作
* 格式： CLEAN 缓存key 元数据文件大小 响应数据文件大小
* 用途： 跟踪已经成功发布的缓存项，可以读取

##### DIRTY操作
* 格式: DIRTY 缓存key
* 用途：跟踪正在被创建或更新的缓存项

##### READ操作
* 格式：READ 缓存key
* 用途：跟踪正在被读取的缓存项

##### REMOVE操作
* 格式：REMOVE 缓存key
* 用途：跟踪已删除的缓存项


#### 操作逻辑

##### 写入缓存
1. 日志中写入DIRTY操作记录
2. 分别将元数据、响应数据写入临时文件中
3. 若写入成功后，将临时文件重命名成正常文件，日志中写入CLEAN操作记录
4. 若写入失败或中止，将临时文件删除，日志中写入REMOVE操作记录

##### 读取缓存
1. 查找缓存项
2. 生成Snapshot对象
3. 日志中写入READ操作记录

##### 删除缓存
1. 查找缓存项
2. 删除数据文件
3. 删除缓存项
3. 日志中写入REMOVE操作记录

##### 重建日志文件
1. 将数据写入临时日志文件中
    a. 写入头信息
    b. 遍历缓存项，正在写入的缓存项，写入DIRTY操作。不在写入的缓存项，写入CLEAN操作
2. 将原日志文件保存为备份文件
3. 将临时日志文件重命名为正式的
4. 删除备份的日志文件

##### 读取日志文件
1. 读取头信息，检查是否正确
2. 发现REMOVE记录，删除指定的缓存项
3. 发现CLEAN记录，创建缓存项，记录数据大小，标记为可读
4. 发现DIRTY记录，创建缓存项，标记为写入中
5. 删除所有正在写入中的缓存项和数据文件



### 缓存策略

#### 判断缓存过期

##### Expires
## 特点
* 支持SPDY，HTTP2
* 支持连接池
* 支持GZIP压缩
* 支持缓存响应
* 使用Interceptors来处理请求和响应


## 关键类

* OkHttpClient  对外接口类
* Request 请求数据类
* Request.Builder  请求生成器
* Dispatcher 请求分发器（异步，同步执行请求）
* Interceptor 拦截器接口类
* Route 路由信息
* Address  地址信息


## 模块

### 接口
* Call
* OkHttpClient
* Dispatcher
* Request

### 协议
* HTTP
* TLS
* WebSocket

### 连接
* Route
* Dns
* RealConection
* StreamAllocation

### 缓存
* DiskLRUCache
* CacheStrategy

### IO
* Http1Codec
* Http2Codec
* okio

## 运行机制

### 分发器
以下几个类组成

**Call**
请求调用器接口类，支持同步调用，异步调用

**RealCall**
请求调用类，实现了Call接口

**Dispatcher**
请求分发器，调度异步运行的请求
使用线程池来运行异步请求



### 拦截器
以下几个类组成

**Interceptor**
拦截器接口类，所有拦截器都需要实现 Response intercept(Chain chain) 接口
从拦截器链中生成响应

**Interceptor.Chain**
拦截器链接口类，对应的实现类为RealInterceptorChain
拦截器可以使用该接口，进入到下一级处理

**RealCall.getResponseWithInterceptorChain**
启动拦截器链
逻辑：
1. 构建一个拦截器堆栈
2. 生成RealInterceptorChain对象
3. 调用链的proceed接口，开始处理请求

**RealInterceptorChain.proceed**
逻辑：
1. 检查拦截器是否已遍历完成
2. 检查运行状态是否正常
3. 从堆栈中获取下一个拦截器
4. 调用其intercept接口

拦截器堆栈
1. 用户自定义的全局拦截器
2. RetryAndFollowUpInterceptor   处理错误和重定向
3. BridgeInterceptor   用于桥接应用层和网络层
4. CacheInterceptor  处理缓存
5. ConnectInterceptor  生成新连接
6. 用户自定义的网络拦截器
7. CallServerInterceptor  访问远端服务器


## 缓存机制

* 将请求url进行md5，用于缓存文件定位
* 定时进行缓存清理
* 使用操作日志来保证缓存的完整性

### 模块
* CacheInterceptor 实现主逻辑
* Cache                  实现缓存功能 
* DiskLruCache    实现缓存文件的操作
* FileSystem       包装文件系统
* CacheStrategy  实现缓存策略


### 数据保存
* 每个url对应一组文件，使用url的MD5值作为文件名
* 缓存文件有两个类型：元数据（meta-data）,  响应数据（body）
* 元数据是 Cache.Entry对象的序列化

### 缓存清理
逻辑：DiskLruCache.cleanupRunnable
1. 判断是否可以运行
2. 缩减缓存文件所占的空间 trimToSize()
3. 缩减日志文件所占的空间 rebuildJournal()


### 操作日志
* 名字：journal
* 格式：文本，按行分隔
* 内容：头信息，操作记录

#### 头信息
* 识别符 libcore.io.DiskLruCache
* 版本号
* 应用版本号
* 操作记录个数

#### 操作记录
 **CLEAN操作**
   * 格式： CLEAN 缓存key 元数据文件大小 响应数据文件大小
   * 用途： 跟踪已经成功发布的缓存项，可以读取

 **DIRTY操作**
   * 格式: DIRTY 缓存key
   * 用途：跟踪正在被创建或更新的缓存项

**READ操作**
 * 格式：READ 缓存key
 * 用途：跟踪正在被读取的缓存项

**REMOVE操作**
 * 格式：REMOVE 缓存key
 * 用途：跟踪已删除的缓存项

#### 操作逻辑

##### 写入缓存
1. 日志中写入DIRTY操作记录
2. 分别将元数据、响应数据写入临时文件中
3. 若写入成功后，将临时文件重命名成正常文件，日志中写入CLEAN操作记录
4. 若写入失败或中止，将临时文件删除，日志中写入REMOVE操作记录

##### 读取缓存
1. 查找缓存项
2. 生成Snapshot对象
3. 日志中写入READ操作记录

##### 删除缓存
1. 查找缓存项
2. 删除数据文件
3. 删除缓存项
3. 日志中写入REMOVE操作记录

##### 重建日志文件
1. 将数据写入临时日志文件中
    a. 写入头信息
    b. 遍历缓存项，正在写入的缓存项，写入DIRTY操作。不在写入的缓存项，写入CLEAN操作
2. 将原日志文件保存为备份文件
3. 将临时日志文件重命名为正式的
4. 删除备份的日志文件

##### 读取日志文件
1. 读取头信息，检查是否正确
2. 发现REMOVE记录，删除指定的缓存项
3. 发现CLEAN记录，创建缓存项，记录数据大小，标记为可读
4. 发现DIRTY记录，创建缓存项，标记为写入中
5. 删除所有正在写入中的缓存项和数据文件



### 缓存策略

#### 判断缓存过期

##### Expires
* 格式：Expires: Thu, 12 Jan 2017 11:01:33 GMT  
* 操作：检查当前时间是否超过该时间，超过即缓存过期  
* 缺点：需要服务器和客户端时间同步

##### CacheControl
* 格式：Cache-Control: max-age=31536000, public
* 操作：检查该文件是否已缓存了max-age时间。如果超过即缓存过期

#### 带条件请求
**缓存过期后，缓存策略应由服务器来判断，客户端需要发送带条件get请求**

##### 请求条件

###### Last-Modified-Date
* 如果第一次请求，服务器返回 Last-Modified: Tue, 12 Jan 2016 09:31:27 GMT，  
* 则再次请求时要带 If-Modified-Since: Tue, 12 Jan 2016 09:31:27 GMT

###### ETag
* 如果第一次请求，服务器返回 ETag: "5694c7ef-24dc"  
* 则再次请求时要带 If-None-Match:"5694c7ef-24dc"

##### 响应
* 如缓存可用，则返回304  
* 如缓存不可用，则返回数据


```flow
start=>start: 浏览器请求
haveCache=>operation: 有缓存
req=>operation: 向服务器请求
reqNoneMatch=>operation: 向服务器请求，带if-None-Match
reqModifiedSince=>operation: 向服务器请求，带if-Modified-Since
resp=>operation: 请求响应，缓存协商
readCache=>operation: 从缓存读取
cacheExpire=>condition: 是否过期？
existsEtag=>condition: 是否存在Etag?
existsLastModified=>condition: 是否存在Last-Modified?
is200=>condition: 响应码是否是200？
is304=>condition: 响应是否是304？
end=>end: 展现

start->haveCache->cacheExpire
cacheExpire(no)->readCache->end
cacheExpire(yes)->existsEtag(yes)->reqNoneMatch
existsEtag(no)->existsLastModified(yes)->reqModifiedSince
existsLastModified(no)->req->resp->end
reqNoneMatch->is200
reqModifiedSince->is200
is200(yes)->resp->end
is200(no)->is304
is304(yes)->readCache->end

```


## 连接池

### 概述
* 使用引用计数来跟踪socket使用
* 自动回收空闲连接

### 模块
* StreamAllocation  实现connection的引用次数
* ConnectionPool  实现Socket连接池，管理、回收连接缓存
* RealConnection   表示一个连接
* ConnectInterceptor  使用连接池获取连接

### 逻辑

#### ConnectInterceptor.intercept()
逻辑：
1. 查找连接 StreamAllocation.findHealthyConnection
2. 从stream中生成HttpCodec对象


#### StreamAllocation.findConnection()
逻辑：
1. 检查stream的状态是否正确
2. 检查当前连接是否可以重用
3. 从连接池中查找连接，调用ConnectionPool.get()
4. 获取一个路由选择器RouteSelector
5. 从连接池中查找连接，带路由信息
6. 创建一个新的连接对象，加入连接池


#### ConnectionPool.get()
* 逻辑：依次判断连接是否和当前地址匹配
* 精确匹配：
    1. 连接要能分配stream
    2. 连接的路由地址要和当前地址匹配，包括host

* 模糊匹配：
    1. 连接要能分配stream
    2. 连接的路由地址要和当前地址匹配，不包括host
    2. 连接需要是http2类型的
    3. 需要传入路由信息
    4. 代理类型要是DIRECT
    5. socket地址需要一致
    6. 端口号要一致



### 地址对象

#### Address
 * 指向服务器的地址
 * 在RetryAndFollowUpInterceptor.intercept()中，使用HttpUrl的信息生成
 * 包括：HttpUrl ， Dns,  支持的协议，ProxySelector


#### RouteSelector
* 用于选择一个合适的路由
* 在RetryAndFollowUpInterceptor.intercept()中，使用Address的信息生成

#### Route
* 表示一个到服务器的具体地址
* 包括：Address， Proxy, InetSocketAddress
* 由RouteSelector在需要时生成