


# Shell语法


## 总体

* 第一行用于指定解释器 #!/bin/sh
* 注释使用 **#**


## 变量

* 赋值：  变量名 = 值
* 取值：  ${变量名}  或  $变量名


## 命令行

### 参数
 * $0   程序名字
 * $1   第一个参数
 * $2   第二个参数
 * $#   总的参数数目
 * $@ 或 $*  参数本身的列表
 * “$@”    参数被解析成一个数组
 * "$*"       参数被解析成一个字符串
 * $?         显示最后命令的退出状态


## 字符串

### 单引号
* 不能使用变量
* 无法使用转义符

```shell
str='this is a string'
```

### 双引号
* 可以使用变量
* 可以使用转义符

```shell
your_name='runoob'
str="Hello, I know you are \"$your_name\"! \n"
```

### 操作
* ${str/a/b}   把a替换成b，只进行一次
* ${str//a/b}  把a替换成b，替换所有
* ${#str}        获取长度
* ${str:pos:length}    获取子字符串


## 数组

* **只支持一维数组**
* 定义：arrayName=(value1 value2 value3 valueN)
* 读取：arrayName[index]    index从0开始，index为@表示获取所有元素
* 长度：#arrayName[@]  或  #arrayName[*]


## 运算符

### 算术运算符

* 使用 expr 命令

```shell
a=10
b=20

val=`expr $a + $b`
echo "a + b : $val"

val=`expr $a - $b`
echo "a - b : $val"

val=`expr $a \* $b`
echo "a * b : $val"

val=`expr $b / $a`
echo "b / a : $val"

val=`expr $b % $a`
echo "b % a : $val"
```

### 关系运算符

* **只支持数字**

| 运算符   | 说明    |   例子   |
| :---- | :----| :------: |
| -eq    | 是否相等 | [ \$a -eq \$b ]  |
| -ne   |  是否不相等 |  [ \$a -ne $b ]  |
| -gt  |  左边是否大于右边 |  [ \$a -gt $b ]  |
| -lt   |  左边是否小于右边 |  [ \$a -lt $b ]  |
| -ge   |  左边是否大于等于右边 |  [ \$a -ge $b ]  |
| -le   |  左边是否小于等于右边 |  [ \$a -le $b ]  |


### 逻辑运算符
| 运算符   | 说明    |   例子   |
| :---- | :----| :------: |
| &&   | AND,与 | [ \$a -lt 100 && \$b -gt 100 ]  |
| \|\|   |  OR,或 |  [ \$a -lt 100 \|\| \$b -gt 100 ]  |


### 字符串运算符
| 运算符   | 说明    |   例子   |
| :---- | :----| :------: |
| = | 是否相等 |  [ \$a = \$b ]  |
| != | 是否不相等 |  [ \$a != \$b ]  |
| -z | 长度是否为0 |  [ -z \$a ]  |
| -n | 长度是否不为0 |  [ -n \$a ]  |
| \$ | 是否为空 |  [ \$ \$a ]  |


### 文件测试运算符
| 运算符   | 说明    |   例子   |
| :---- | :----| :------: |
| -d  | 是否是目录 | [ -d \$file ] |
| -f  | 是否是普通文件 | [ -f \$file ] |
| -r | 文件是否可读 | [ -r \$file ] |
| -w | 文件是否可写 | [ -w \$file ] |
| -x | 文件是否可执行 | [ -x \$file ] |
| -e | 文件、目录是否存在 | [ -e \$file ] |
| -s | 是否不为空（文件大小大于0） | [ -s \$file ] |



## 流程控制

### if语句
```shell
 if condition 
 then
    command
 elif condition  
 then
    command
 else
    command
 fi

#一行格式
if  ... ; then ... ; fi
```

### case语句

* **匹配字符串**

```shell
case value  in
   pattern1 ) command  ;;
   pattern2 ) command  ;;
esac
```

### 循环语句
```shell
 while  condition 
 do
    command
 done
 
 for var in item1 item2 itemN  do
    command           
 done

 #一行格式
 for var in item1 item2 itemN ; do command ; done
```

## 函数

* 获取参数使用 \$1 , \$2 ...
* 返回值使用 return  (值为 0 -- 255)
* 返回内容使用 echo xxx

### 定义
```shell
function name()
{
	code
}

funName() {
	code
}
```


## 重定向

| 运算符   |  说明    |  
| :----  | :---- |
| cmd > file |  将输出重定向到文件 |
| cmd < file | 将输入重定向到文件 |
| cmd >> file | 以追加方式重定向输出到文件 | 

### 类型
* 标准输入(stdin)，文件描述符为 0
* 标准输出(stdout)，文件描述符为  1
* 标准错误(stderr)，文件描述符为 2 
* /dev/null ，内容会被丢弃

```shell
command 2 >> file   #重定向错误信息
command > file 2>&1    #将stdout, stderr合并重定向
command > /dev/null    #不显示任何输出
```


## 命令

### echo
* **用于字符串输出**

### printf
* **格式化符号类似于C**
* 格式：printf format [arguments]

### test
* **用于检测某个条件是否成立**