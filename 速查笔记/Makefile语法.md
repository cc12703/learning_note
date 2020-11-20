

# Makefile语法

## 要点
* \ 为换行符、转义符
* 通配符为  *，？，[…]
* makefile中只应该有一个最终目标


## 规则
### 格式
> target : prerequisites1   prerequisites2
>    command1
>    command2
    
#### 说明
* target  目标文件
* prerequisites  生成target所要的文件
* command   make所要执行的命令(任意的shell命令)

## 执行过程
1. 读入所有的makefile
2. 读入被 include 的其他makefile
3. 初始化文件中的变量
4. 分析所有的规则
5. 为所有的目标文件，创建依赖关系链
6. 确定哪些目标要重新生成
7. 执行生成命令


## 变量
* variable name :=  variable value
    * 定义一个简单扩展变量，右值部分会被立刻扩展

* variable name = variable value
    * 定义一个递归扩展变量，每次变量被使用时，都会进行一次扩展

* variable name ?=  variable value
    * 定义一个条件变量，当该变量的值不存在时，才会被赋值

* variable name +=  variable value
    * 将值附加到变量中


## 自动变量
* \$@  工作目标的文件名
* \$*  工作目标的主文件名
* \$%  库文件中的文件名元素
    * 语法  archive(member)
    * 若工作目标为 foo.a(bar.o) 则 $%为 bar.o
* \$< 第一个必要条件的文件名
* \$? 所有时间戳在工作目标之后的必要条件，以空格相隔开
* \$^ 所有必要条件的文件名，以空格相隔开



## 函数

### 自定义宏
> define  macro-name
>    \$1  ----  第一个参数
>    \$2  ----  第二个参数

### 调用自定义宏
>  \$(call macro-name, param1 param2 paramN)

### 调用内置函数
> \$(function-name  arg1, arg2, argN)


### 字符串函数

* $(filter pattern …. , text)
    * 功能：返回与pattern相符的text
    * text 为一系列被空格隔开的单词
    * pattern 为模式，%表示匹配任意个字符