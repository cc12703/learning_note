[TOC]


# Pandas

## 总体

* 处理结构化数据的库
* 基于NumPy构建的


## 数据结构

### Series

* 类似于一维数组的对象
* 包括 一组数据 和 一组对应的标签（索引）

#### 用法

##### 创建对象
```py
    obj = Series([4,3,1,3])  #自动创建0 - N-1的整数型索引
    obj = Series([4,3,1,3], index=['d','e','a','c'])
```

##### 操作对象
```py
    obj.values #获取数组表示形式
    obj.index #获取索引对象
    obj[obj > 0]   #过滤，获取所有大于0
    obj * 2  #所有值都乘以2
    obj1 + obj2  #将两个对象相加，自动对齐不同索引的数据
```

##### 数据转换
* 使用map函数，传入一个函数、字典对象

```py
data['animal'] = data['food'].map(str.lower).map(meat_to_animal)
```


### 索引对象

* 负责管理轴标签和其他元数据

#### 类型
* Index 最泛化的索引
* MultiIndex 层次化索引

#### 方法
* diff 计算差集
* union 计算并集
* intersection 计算交集


### DataFrame

* 一个表格型的数据结构
* 包含一组有序的列
* 每列值可以是不同类型
* 既有行索引，也有列索引


#### 基本操作

##### 创建对象
* 使用等长的字典（传入列名，索引名）
* 使用嵌套字典（外层键做列，内存键作为索引）

```py
    #传入由等长列表组成的字典
    data = {'state':['a','b'],
            'year':[2000,2001],
            'pop':[1.5,1.0]}
    frame = DataFrame(data) #键名为列名
```

##### 选取

* obj[val] obj[v1 : v2] 选取单个列，一组列
* obj.ix[val] obj.ix[v1 : v2] 选取单个行，一组行
* obj.ix[val1, val2] 同时选取行和列
* obj.xs方法，根据标签选择行或列，返回一个Series

```py
    frame['state']
    frame.state
```

##### 映射

* 使用apply将函数应用到由各行，各列所形成的一维数组上

```py
    f = lambda x: x.max() - x.min()
    frame.apply(f)  #应用到各列所形成的数组上
    frame.apply(f, axis=1)   #应用到各行所形成的数组上
```

* 使用applymap将函数应用到各个元素上

```py
    f = lambda x: '%.2f' %x
    frame.applymap(f)
```


## 基本操作

### 数据加载

#### read_csv 
* 从文件中加载带分隔符的数据。默认为逗号
* 默认使用第一行的数据作为列名
* 使用参数header来指定行号
* 使用参数names来设置列名

#### to_csv
* 将数据写入到逗号分隔的文件


#### sparse
* 转换成稀疏类型 astype(pd.SparseDtype())
* 获取稀疏矩阵 sparse.to_coo() 类型为 scipy.sparse.spmatrix


##### 转换成其他格式
* tocoo()
* tocsc()
* tocsr()


### 数据规整化


#### 合并数据

##### pd.merge
* 根据一个/多个键，将不同的DataFrame中的行连接起来

###### 参数
* on 键名，单个或者列表
* how 连接方式，inner, left, right, outer


#### 重复数据
* drop_duplicates()，移除重复行，可以指定部分列
* duplicated(), 返回一个布尔型的Series，表示各行是否是重复行


#### 数据映射
* 使用map()映射指定列的数据

```py
   data['animal'] = data['food'].map(str.lower).map(meat_to_animal)

   data['animal'] = data['food'].map(lambda x: meat_to_animal[x.lower()])
```

#### 替换值
* fillna()填充缺失的数据
* replace() 替换指定的数据

```py
   data.replace(-999, np.nan)
   data.replace([-999, -10000], np.nan)
   data.replace([-999, -10000], [np.nan, 0])
   data.replace({-999: np.nan, -10000: 0})
```


#### 过滤异常值
```py
   #过滤指定列
   col = data[3]
   col[np.abs(col) > 3]

   data[(np.abs(data) > 3).any(1)]
```


#### 离散化
* 将连续数据拆分成面元（bin）

##### pd.cut

###### 参数
* bins 面元数量，面元边界
* labels 面元名称
* precision 划分精确度
* duplicates = drop 丢弃重复的面元边界

###### 返回
* 数据类型是category，可以使用astype()转换成需要的类型


##### pd.qcut
* 根据样本的分位数进行划分
* 得到大小基本相等的面元

###### 参数
* q 分位数，分位边界


### 分组聚合




### 其他

#### 透视表（pivot_table）
* 根据一个，多个键对数据进行聚合
* 根据行、列上的分组键将数据分配到各个矩形区域汇中

##### 函数说明
![](https://gitee.com/cc12703/figurebed/raw/master/img/20201109183034.png)

##### 参数
* values 待聚合的列名字（可以多个）
* index  用于分组的列名（可以多个），出现在透视表的行
* columns 用于分组的列名（可以多个），出现在透视表的列表
* aggfunc 聚合函数，默认为mean
* fill_value 填充缺失值
* margins 是否添加行/列小计和总计


## 绘图

### 设置

#### 显示中文
```py
import matplotlib as mpl
mpl.rcParams['font.sans-serif'] = ['KaiTi']
mpl.rcParams['font.serif'] = ['KaiTi']
```

### 函数

#### plot
* 用于给DataFrame, Series生成图表
* 对于Series 索引会被用于绘制x轴
* 对于DataFrame 索引会被用于绘制x轴，每行的所有列会被分为一个组


### 图形

#### 柱状图
* 每行的数据被单独绘制，用于离散数据
* 使用plot函数
* 参数 kind=bar，barh(水平柱状图)
* 参数 stacked=True，每行所有列的值会被堆积在一条柱上

#### 直方图
* 对值频率进行离散化显示的柱状图
* 数据点被拆分到离散的、均匀的面元中
* 绘制各个面元中数据点的数量
* 使用hist函数


#### 密度图
* 用于绘制核密度估计
* 使用plot函数
* 参数 kind=kde


#### 散布图
* 用于观察两列数据之间的关系
* 使用plt.scatter函数
* 使用DataFrame.scatter_matrix函数绘制散布图矩阵





                                                                                                 