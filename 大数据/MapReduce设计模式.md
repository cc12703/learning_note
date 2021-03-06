
# MapReduce设计模式

[TOC]


## Hadoop MapReduce介绍

### 总体
* 作业被分成一系列运行在分布式集群中的map任务和reduce任务
* 每个任务都工作在被指定的小的数据集上
* map任务：数据载入、解析、转换、过滤
* reduce任务：对map任务的中间数据进行分组和聚合操作

### map任务
* 分成四个阶段：reader,  mapper,  combiner,  partitioner
* 输出会被发送到reducer做后续处理

#### reader
* 将输入数据解析成记录，转换成key / value 对的形式
* 不负责解析解析记录本身

#### mapper
* 用户通过自定义的map代码来处理输入的 key / value 对
* key 是在reducer中被分组的依据
* value 是reducer需要分析的数据

#### combiner
* 一个可选的本地reducer，用于在map阶段聚合数据
* 在大部分情况下，可以减少通过网络传输的数据量

#### partitioner
* 将 key / value 对拆分成分片，每个reducer对应一个分片
* 每个分片数据会被写入本地文件系统，等待被reducer拉取

#### mapper参数
* 输入键
* 输入值
* 输出键
* 输出值类型


### reduce任务
* 分成四个阶段：shuffle,  sort,  reducer,  output

#### shuffle和sort
* 拉取map阶段生成的输出文件到本地机器
* 将数据按键排序并写到一个数据列表中
* 排序的目的是将数据按键聚合在一起

#### reducer
* 处理已分组的数据
* 依次为每个键对应的分组执行reduce函数

#### output
* 将 key / value 对通过record writer 写入到输出文件中

#### reducer参数
* 输入键
* 输入值
* 输出键
* 输出值


## 概要模式

* **对于一份新的数据，概要分析可以评估出哪些指标是有吸引力的**
* 将相似数据进行分组
* 对分组数据进行分析操作

### 数值概要
**是计算数据聚合统计值的一般性模式**

#### 目的
* 基于某个键将记录分组
* 对每个分组计算一系列的聚合值
* 得到较大数据集的高层次视图

#### 场景
* 要处理的是数值或者计数
* 数据可以按特定字段分组

#### 结构

mapper  -- (键，汇总字段) --> partitioner  ---> reducer (分组B，汇总)

##### mapper
* 输出键需要包含进行分组操作的所有字段
* 输出值包含所有相关的数值字段

##### combiner
* 极大地减少通过网络传输到reduce的数据
* 统计函数必须要满足 结合律 和 交换律

##### partitioner
* 通过定制来实现更优的数据分片

##### reducer
* 对接收到的数据执行统计函数

#### 结果
* 输出由一系列part文件组成
* 每个reducer的输入分组对应一条记录
* 每条记录包括reducer键和对应的聚合值

#### 应用
* 单词计数
* 记录计数  （按每天、每周、每小时来计算）
* 最大值、最小值、计数
* 平均值、中位数、标准差

#### 性能
* 需要设置合适的reduce数目
* 可能会出现数据倾斜（某个特定键产生了大量的key/value对）


### 倒排索引

#### 目的
* 产生一个数据集的索引，提供更快的搜索或数据丰富能力

#### 场景
* 需要快速搜索查询响应

#### 结构

mapper -- (关键字，唯一ID)  --> partitioner  --->  reducer (关键字A， ID列表)

##### mapper
* 输出键包括索引所需要的字段
* 输出值为全局唯一的标识符

##### partitioner
* 如果中间键不是非常分散的话，可以定制该步骤以获取更好的负载均衡

##### reducer
* 将输入键映射到一组唯一的记录标识符的集合

#### 结果
* 输出一系列文件
* 每个文件包括一组映射（字段值 ---> 一系列唯一ID）

#### 性能
* mapper中内容解析的计算成本
* 索引键会存在热点的情况
* 若唯一键的数目和标识符数据都非常大，则需要增加reducer的数目


### 计数器计数

使用mapReduce框架自身的计数器，在map阶段计数一个全局计数

#### 目的
* 得到大数据集计数概要的一种高效方法

#### 场景
* 在一个大数据集上收集计数或汇总
* 需要创建的计数器数目很小（两位数以内）


#### 结构
mapper --  累加计数器A ---> taskTracker  ---> jobTracker
mapper --  累加计数器B ---> taskTracker -------|

##### mapper
* 每次处理每条记录时，对计数器进行累加
* 计数器信息会被上报给 jobTracker 完成聚合


## 过滤模式

通过对数据的相似字段做概要与分组来得到数据的高层次视图

### 过滤

#### 目的
* 过滤掉不感兴趣的记录并保留需要的记录

#### 场景
* 数据可以被解析成 记录
* 可以通过特定的准则来判断记录是否保留

#### 结构
**不需要reduce阶段，单独处理每一条记录**

输入  --->   mapper(输出)  -->  过滤器

##### mapper
* 对每一条记录执行评估函数
* 如果评估函数返回true, 则输出键和值

#### 结果
* 输出一个已选择过的记录子集

#### 已知应用
* 近距离观察数据：选择一个特定的数据子集
* 跟踪某个事件的线索
* 分布式grep
* 数据清理
* 简单随机抽样
* 移除低分值数据

#### 性能
* **注意输出文件的大小和数目**
* 每个mapper都会输出一个文件
* 可能会输出大量的小文件


### 布隆过滤

#### 目的
* 使用一个预定义值集合来过滤记录
* 对每条记录，可以抽取其中一个特征

#### 场景
* 数据可以被分成多条记录
* 每条记录可以抽取出一个特征，并可以被一系列的热门值表示
* 对于热门值有预先确定的元素集合
* 能接受结果误判的存在


#### 结构
* 分成两个步骤：训练布隆过滤器，过滤操作
* 过滤过程中使用分布式缓存

##### 训练
* 使用一个特定值的列表
* 加载数据，将记录加入布隆过滤器

##### 过滤
* 加载布隆过滤器
* 遍历记录并检查是否在布隆过滤器中


#### 已知应用
* 移除大多数不受监控的值
* 对成本很高的集合成员资格检查做预先过滤


### Top10

#### 目的
* 不使用全局排序，而是找出有限数目的高价值记录

#### 场景
* 需要一个比较函数
* 输出记录数相对于输入记录数是异常的小

#### 结构
mapper  --- 找出本地top10  ---> reducer (找出最终的top10)
mapper  --- 找出本地top10  ------|

##### mapper
* 找出本地的top K
* 使用null键将top K记录发出

##### reducer
* 只有一个reducer
* 找出最终的top K

#### 已知应用
* 异类分析
* 选取感兴趣的数据
* 指标面板

#### 性能
* 只能使用一个reducer
* 该reducer需要处理 K x M 条记录（K是mapper任务输出的记录数，M是mapper任务数）
* 当K值变大时，该模式会变得低效
* 该模式只适用于K值相对较小的情况 （几十或几百）


### 去重

#### 目的
* 对一系列相似记录，找出唯一集合

#### 场景
* 数据集要存在重复值

#### 结构
* 使用了mapReduce框架将相同键进行分组来实现去重

##### mapper
* 提取需要去重的字段
* 输出字段对应的记录作为键，null作为值

##### reducer
* 简单的输出键

#### 结果
* 输出的记录保证是唯一的
* 顺序是乱的

#### 已知应用
* 数据去重
* 抽取重复值
* 规避内连接导致的数据膨胀

#### 性能
* 需要考虑reducer的数目
* 使用combiner的本地去重来优化网络IO


## 数据组织模式


### 分层
**从数据中创建出不同于原结构的新记录**

#### 目的
* 将基于行的数据转化成分层的格式（json, xml）

#### 场景
* 数据源被外键链接
* 数据是结构化的并且是基于行的

#### 结构

数据集A  ---  数据集A的mapper  ---->  
数据集A  ---  数据集A的mapper  ---->                             ---  层次化的reducer  --->  结果
                                                               混排 和 排序   
数据集B  --- 数据集B的mapper  ---->                               ---  层次化的reducer  --->  结果
数据集B  --- 数据集B的mapper  ---->

使用MultipleInputs 给不同的数据源指定mapper类

##### mapper
* 将数据解析成更简洁的形式，便于reducer处理
* 输出键用于确定如何分层
* 输出值中需要记录其来源

##### reducer
* 根据已分组数据来建立分层数据结构

#### 已知应用
* 预连接数据
* 为HBase或MongoDB准备数据

#### 性能
* 需要关注 mapper发送了多少数据给reducer
* 需要关注 reducer 构建时占用了多少内存


### 分区
将数据进行分类，但不关心记录的顺序

#### 目的
* 将数据集中相似的记录分成不同的、更小的数据集

#### 场景
* 必须提前知道有多少个分区

#### 结构
特点：数据是通过分区器进行分区的

输入  ---  id mapper --->  partitioner  ---->  
输入  ---  id mapper --->  partitioner  ---->                               ----  id reducer  ----> 分区A
                                                                        混排 和 排序 
输入  ---  id mapper --->  partitioner  ---->                              ----  id reducer  ----> 分区A
输入  ---  id mapper --->  partitioner  ---->  

##### mapper
* 大多数情况下，都使用相同的mapper

##### partitioner
* 决定每条记录发送到哪个reducer
* 每个reducer对应哪个特定分区

##### reducer
* 大多数情况下，使用identity reducer即可

#### 结果
* 每个分区对应一个输出part文件
* 文件比较大，优先考虑使用基于块压缩的SequenceFile来存储数据

#### 已知应用
* 按连续值裁剪分区
* 按类别裁剪分区
* 分片

#### 性能
* 需要关注每个分区是否有相似数量的记录
* 如果分个分区特别大，则需要拆分成更小的分区



### 分箱

#### 目的
* 将数据集中每条记录归档到一个、多个类别

#### 结构
* 特点：在map阶段对数据进行拆分
               
输入   ----   分箱mapper  --->  part A, part B, part C

##### mapper
* 使用了MultipleOutput来输出多个文件
* 对于每条记录，会遍历每个箱子的标准列表

#### 结果
* 每个mapper将为每个箱子输出一个小文件
* 需要将小文件合并成较大一点的文件



### 全排序

#### 目的
* 希望数据能够按照指定的键进行并行排序

#### 结构
* 特点：排序前要先划分一组分区，且每个分区要有相同规模的数据子集
* 分为两个阶段：分析、排序

##### 分析
* 需要对数据进行随机采样，对随机样本进行分区
###### mapper
* 进行简单的随机采样
* 输出键为排序键
* 输出值为null

###### reducer
* 只需要一个reducer
* 将排序键收集到一个已排序的列表中

##### 排序
###### mapper
* 提取排序键，将记录本身作为输出值

###### partitioner
* 加载分区文件，可以使用TotalOrderPartitioner

###### reducer
* 简单的将值输出


#### 性能
* 该操作代价比较高，需要加载和解析数据两次


### 混排

#### 目的
* 将一个给定的记录集完全随机化

#### 场景
* 为了匿名的目的对数据混排
* 为了在随机的数据集中获得可重复的随机采样

#### 结构
##### mapper
* 生成一个随机值
* 输出键为 随机值，输出值为 输入记录

##### reducer
* 按照随机键对记录进行排序

#### 性能
* 该模式有很好的性能特性
* reducer越多，数据的展开越快