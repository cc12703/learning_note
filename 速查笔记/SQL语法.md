

# SQL语法

## 概述

* 结构化的查询语言、用于访问数据库
* 对大小写不敏感、使用 单引号 引用 文本值

### 数据库结构
* 一个数据库包含 多个表
* 一个表包含 名字标识 + 多个记录(行)
* 一个记录包含 多个数据项(列)

### 语句种类

#### DML(数据操作语句)
* select      从表中获取数据
* update    更新表中的数据
* delete     从表中删除数据
* insert      向表中插入数据

#### DDL(数据定义语句)
* create database    创建新数据库
* alter database       修改数据库
* create table          创建新表
* alter table             修改表
* drop table             删除表
* create index         创建索引
* drop  index           删除索引


## 数据类型

### sqlite
**使用动态数据类型，会根据存入值自动判断**

* null  空值
* integer  带符号的整型
* real  浮点数
* text  字符串文本, 存储可变长度的非Unicode数据
* ntext 字符串文本, 存储可变长度的Unicode数据
* blob   二进制对象
* 布尔类型
    * 使用整型代替
    * 1表示true，0表示false
* 日期时间类型
    * text形式:   "YYYY-MM-DD HH:MM:SS.SSS"
    * real形式:    以Julian日期格式存储
    * integer形式: 以从1970-01-01 00:00:00到当前时间所流经的秒数

### MySQL

#### 文本类型
* char(size)   固定长度的字符串(最多255个字符)
* varchar(size)  可变长度的字符串(若大于255个字符，则转换为text类型)

* text         最大长度为 65,535 个字符的字符串
* mediumtext    最大长度为 16,777,215 个字符的字符串
* longtext     大长度为 4,294,967,295 个字符的字符串

* blob     存放最多 65,535 字节的二进制数据
* mediumblob    存放最多 16,777,215 字节的二进制数据
* longblob     存放最多 4,294,967,295 字节的二进制数据

#### 数字类型
* tinyint(size)    范围  -128 到 127 ，8位
* smallint(size)   范围  -32768 到 32767，16位
* int(size)        范围  -2147483648 到 2147483647，32位
* float(size,d)    带有浮动小数点的小数字
* double(size,d)   带有浮动小数点的大数字

#### 日期时间类型
* data 
    * 格式  YYYY-MM-DD
    * 范围  '1000-01-01' 到 '9999-12-31'
* datatime  
    * 格式  YYYY-MM-DD HH:MM:SS
    * 范围  '1000-01-01 00:00:00' 到 '9999-12-31 23:59:59'
* timestamp  时间戳，从'1970-01-01 00:00:00'至今的秒数


## 语句

### select

#### 要点
* 从表中选取数据，并存入一个结果表中
* 列名集：列名1，列名2，列名3
* \*  表示所有列

#### 写法
* 常规:   select 列名集 from 表名
* 去除重复值: select  distinct  列名集  from  表名   
* 有条件的从表中选取数据: select  列名集  from 表名   where 列名  运算符  值    
* 对结果进行排序: select 列名集  from 表名  order by  列名集  

#### sql函数
* 写法：select   function(列名)   from  表名

##### 函数
* count  返回某列的行数，不包含NULL值
* count(\*)   返回被选的行数
* avg      返回某列的平均值
* sum    返回某列的总和
* max    返回某列的最大值
* min    返回某列的最小值


### insert
* 向表中插入新的行

#### 写法
* 插入新行:  insert into 表名 values  (列值1，列值2，…..) 
* 插入指定列数据: insert into  表名  (列名1，列名2，…..) values (列值1，列值2，…..)


### update
* 修改表中的数据，某行中的多个列

#### 写法
> update 表名  set 列名 = 列值，列名 = 列值,  ….   where 列名 = 列值


### delete
* 删除表中的行

#### 写法
* 删除表中的指定行: delete from 表名  where 列名 = 列值
* 删除表中的所有行: delete  from 表名


 ## join操作

 ### 要点
 * 用于从多个表中获取数据
 * 数据库中表使用 键 进行相互联系，将表间的数据交叉捆绑在一起
 * 主键(Primary Key），表中的一列，每行的值都是唯一的
