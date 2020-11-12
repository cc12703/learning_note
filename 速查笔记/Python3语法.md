
# Python3语法


## 基础

### 编码
* 源码文件以 UTF-8 编码
* 字符串以 unicode 编码


### 注释
* 单号 使用 **#**
* 多行 使用 **'''**, **"""**

### 三元操作符
* X if C else Y

```python
smaller = x if x < y else y
```



## 数据结构

### 序列类型
* 包括：字符串、列表、元组

#### 通用

##### 操作符
* seq [ index ]
* seq [ index1 : index2 ]  切片操作，index1包含，index2不包含
* seq * expr   序列重复expr次
* seq1 + seq2  连接序列
* obj  in seq  判断对象是否在序列中
* obj not in seq 判断对象是否不在序列中


##### 内建函数
* len ( seq ) 返回seq长度
* max ( iter,  key )  返回序列中的最大值，key会传给sort函数
* min ( iter, key)   返回序列中的最小值，key会传给sort函数
* sorted ( iterable, key, reverse)  序列排序，返回一个新序列
* reversed（iterale）序列反转，返回一个迭代器
* sum ( seq, init)  计算序列和，等于 reduce ( operator.add, seq, init )
* enumerate ( seq )  返回enumerate对象，每个元素是一个元组 （index,  item）
* zip（iterable, ...）将多个序列对象，打包成一个个元组，返回一个迭代器


**zip**
```python
a = [1,2,3]
b = [4,5,6]
c = list(zip(a, b))
# c 为 [(1,4), (2,5), (3,6)]
```


#### 字符串

##### 格式化

**%-formatting**
```python
 name = 'Eric'
 age = 74
 "hello, %s, You are %s" % (name, age)
```

**str.format()**
```python
name = 'Eric'
age = 74
"hello {}, You are {}".format(name, age)
```

**f-string**
```python
name = 'Eric'
age = 74
f"hello {name}, You are {arg}"

f"{2 * 37}"

name = 'Eric'
f"{name.lower()} is funny"
```


##### 模板
```python
from string import Template
    s = Template('There are ${howmany} ${lang} Quotation Symbols')
    s.substitute(lang='python', howmany=3)
```

##### 原始字符串
* 定义：以 r/R 开头
* 所有字符都是直接按照字面意思来使用，没有转义特殊字符


#### 列表

* 使用 [xxx, xxx] 定义

##### 列表解析
* 语法：[expr for iter_var in iterable]

```python
[ i * 2 for i in [8, -2, 5]] --> [16, -4, 10]

[ i for i in range(8) if i % 2 == 0] --> [0, 2, 4, 6]
```

#### 元组
* 使用 (xxx, xxx) 定义
* 和列表的区别：不可变型


### 映射类型

* 包括： 字典

#### 创建
```python
dict1 = {}
dict2 = {'name' : 'earth', 'port' : 80}
```

#### 操作符
* dict [ key ] = value  更新、增加条目
* del dict [ key ]   删除条目
* del dict   删除整个字典
* key  in dict   判断条目是否存在
* key not in dict 判断条目是否不存在


#### 内建函数
* len ( dict ) 返回条目个数
* dict.clear()  清除所有条目
* dict.get( key, default )  获取条目值
* dict.has_key( key )  判断条目是否存在
* dict.items()   返回 键、值对元组的列表
* dict.keys() 返回键的列表
* dict.values()  返回值的列表
* dict.update( dict )  将字典的条目加到另一个字典中
* dict.fromkeys( seq, value ) 从列表中创建字典，使用value为默认值


### 集合类型

* 包括：set

#### 操作符
* set()  创建一个可变集合
* frozenset()  创建一个不可变集合
* val in set    判断值是否在集合中
* set1 |  set2  合并两个集合
* set1 & set2  获取两个集合的交集

#### 内建函数
* len ( set ) 返回元素个数
* set.add ( val )  增加元素
* set.remove ( val ) 删除元素
* set.discard ( val ) 若元素在集合中，则删除该元素


## 模块

* 模块：一个扩展名为py的文件
* 包：一个目录，包含__init__.py文件

### 导入语法

#### 导入顺序
1. 标准库模块
2. 第三方模块
3. 自定义模块


#### import
* 加载模块
* 语法：import module



#### from-import
* 从模块中导入指定部分到当前的命名空间
* 语法：from module import name1, name2
* 语法：from module import * (导入模块的所有内容)

#### as 
* 将导入的模块绑定给局部变量
* 语法：import module as var-name
* 语法：from module import name as var-name


#### 导入包

* 使用 点语法

```python
import Phone.Mobile.Analog

from Phone import Mobile
from Phone.Mobile import Analog
```

#### 相对导入
* import 只支持绝对导入
* from-import 支持绝对导入，相对导入
* 使用 点 进行标识

```python
from .Analog import dial
from ..common_util import setup
from ..Fax import G3.dial
```


## 编程范式

### 函数式

#### lambda
* 用于创建匿名函数 
* 语法：lambda [arg1, arg2, xxx] : expression

```python
lambda x, y: x + y

a = lambda x, y=2: x + y
a(3)
a(3, 6)
```

#### 内建函数

##### filter
* 原型：filter (func, seq) 
* 作用：使用func来过滤seq序列，并返回一个序列

```python
filter(lambda n: n%2, [xxx])
```

##### map
* 原型：map (func, seq1, seq2, xxx) 
* 函数定义：func(seq1[i], seq2[i], xxx ) 
* 作用：将func作用于序列的每个元素，并返回一个序列 
* 注意：map会延迟求解。若需要求值，则要加list()

```python
list(map (lambda x: x+2, [0,1,2,3])) -> [2,3,4,5]
map(lambda x, y: x+y, [1,3,5], [2,4,5])
```

##### reduce
* 原型：reduce (func, seq, init) 
* 函数定义：func( last, seq[i] ) 
* 作用：使用func来合并序列，init为初始化值 
* 注意：reduce位于 functools 包中

```python
reduce(lambda x,y: x+y, [0,1,2,3,4], 0)  -> 10
```

### 面向对象

#### 创建类
```python
class Hello(object) :
    foo = 100     #类属性

    def __init__(self) :    #构造函数
        self.data = 1     #实例属性

    def incData(self) :   #实例方法
        self.data = self.data + 1  

    @staticmethod
    def foo() :       #静态方法
        pass

    @classmethod    
    def foo(cls) :   #类方法
        pass 
```

#### 类的内部属性
* C.name    类的名字
* C.doc      类的文档字符串
* C.bases   类的所有父类（元组）
* C.dict  类的属性
* C.module  类所在的模块


## 异常处理

### try-except
```python
try :
    #正常代码
except Exception, reason :
    #异常处理代码
    #reason 是一个异常类的实例
except Exception1 :
    #异常处理代码
except (Exception2, Exception3) :
    #异常处理代码
```

### try-finally
```python
try :
    #正常代码
except :
    #异常处理代码
else :
    #无异常时执行的代码
finally :
    #无论如何都执行的代码
```

### 标准异常
* BaseException 所有异常的基类
* SystemExit   解释器请求退出
* Exception 常规错误的基类
* StopInteration 迭代器没有更多的值
* Warning 警告的基类

### with
* 用于简化资源的分配和释放代码
* 语法 with context_expr [as var] 

```python
with open('/etc/passwd', 'r') as f:
    for eachLine in f:
        #...
```



## 类型

### 类型标注

* 标注参数 使用 **:** 符号
* 标注返回值 使用 **->** 符号
* 标注数据结构 使用typing模块

```python
from typing import List

def plus(a: int, b: int = 2) -> int :
    return a + b

def func(name: str) -> List[str] :
    pass
```


### 类别别名

```python
from typing import List
Vector = List[float]

def scale(scalar: float, vector: Vector) -> Vector:
    return [scalar * num for num in vector]

ConnectionOptions = Dict[str, str]
Address = Tuple[str, int]
Server = Tuple[Address, ConnectionOptions]
```
