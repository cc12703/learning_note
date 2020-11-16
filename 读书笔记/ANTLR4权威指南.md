
[TOC]



# ANTLR4权威指南



## 纵观全局

### ANTLR功能
* 解析由ANTLR语法写成的特定语言的语法文件
* 将其转换为手工构建一般的语法分析器

 
#### 概念
* 语法是一些规则的集合，每条规则描述出一种词汇结构
* 词法符号：包含类型和对应的文本
* 语法分析树：叶子节点是输入的词法符号，内部节点是词组名用于识别子节点
   
#### 流程
* 词法分析将 字符流 解析成 词法符号
* 语法分析将  词法符号 组合成 语法分析树


### 语法分析器

* ANTLR会产生一个递归下降的语法分析器
* 是若干递归方法的集合，每个方法对应一条规则
* 下降过程是指从语法树的根节点开始，朝着叶节点进行解析的过程

```java
// assign: ID = expr ;
void assign() {   //根据规则生成的方法
	match(ID);
	match('=');
	expr();
	match(';');
}
```

### 语法分析树

#### 遍历方法
* 监听器方式（默认模式）
* 访问者模式

#### 监听器
* 每个语法文件会生成一个ParseTreeListener的子类
* 该类中，每条规则会有对应的enter方法和exit方法
* 使用ParseTreeWalker类来遍历listener类
* 优点：一切都是自动进行的

#### 访问者
* 提供一个接口和一个默认实现类
* 需要主动调用visit()方法来进行遍历

### ANTLR运行

#### 模块
* ANTLR工具
* ANTLR运行库

#### 步骤
* 写语法文件 xxx.g4
* 运行ANTLR工具，生成一组文件
* 编译生成的文件，使用grun进行测试
* 和主程序进行集成


#### 生成文件
* XXXParser.java   语法分析器
* XXXLexer.java      词法分析器
* XXX.tokens         词法符号的数字类型
* XXXListener.java   监听器接口
* XXXBaseListener.java  监听器的默认实现



## 设计语法

### 起始规则

* 使用 伪代码 来描述输入文本的整体结构

#### CSV文件
* 文件:  由多个行组成，行由换行符结尾
* 行: 由多个字段组成，字段由逗号分隔
* 字段: 数字 或  字符串


### 语言模式
* 序列
* 选择
* 词法符号依赖（左右括号匹配）
* 嵌套结构


#### 序列模式

##### 固定长度
**例子**
```
retr : 'RETR' INT '\n' ; //匹配 关键字-整数-换行符 序列
```

##### 任意长度
* ” + ”  表示一个或多个
* “ * ”  表示零个或多个
* " ? "  表示零个或一个

**例子**
```
file : (row '\n')* ;
row  : field (',' field)* ;
field : INT ;
```

#### 选择模式
* ” | “ 用于分隔多个可选的语法结构

**例子**
```
field : INT | STRING ;
stmt : node_stmt 
     | edge_stmt
     | attr_stmt
     | id '=' id
     ;
```

#### 词法符号依赖
* 使用一个序列来指明所有配对的符号

**例子**
```
vector : '[' INT+ ']'  ;  //[1], [1 2], [1 2 3]

expr : expr '(' exprList? ')'    //like f(), f(x), f(1,2)
     | expr '[' expr ']'         //like a[i], a[i][j]
     ;
```

#### 嵌套模式
* 是一种自相似的语言结构，即子词组也遵循相同的结构

**例子**
```
//直接递归
stat : 'while' '(' expr ')' stat
     | '(' stat ')'
     ...
     ;

//间接递归
stat : 'while' '(' expr ')' stat
     | block
     ...
     ;
block : '{' stat* '}'
```


### 左递归
* 自顶向下的语法和语法分析器的经典形式是无法处理左递归的

#### 例子
```
//无法处理 1+2*3 的输入，因为会有两种方式可以解释
epxr : epxr '*' expr
     | expr '+' expr
     | INT
     ;
```

#### ANTLR方法
* 优先选择位置靠前的备选分支
* 使用assoc手工指定结合性

**例子**
```
expr : <assoc=right> expr '^' expr
     | INT
     ;
```

### 词法结构

#### 匹配标识符
```
ID : ( 'a'..'z' | 'A'..'Z' )+ ;
ID : [a-zA-Z]+ ;
```

#### 解决字符串常量冲突
1. 从文法规则中选出所有的字符串常量，作为隐式词法规则
2. 将其放在文法规则后，显式词法规则前
3. 词法分析器优先使用位置靠前的词法规则


#### 匹配数字
```
INT : '0'..'9' + ;

//浮点数
FLOAT : DIGIT+ '.' DIGIT+
      |        '.' DIGIT+
      ;

fragment
DIGIT : [0-9] ;   
```

#### 匹配字符串常量
* ？表示非贪婪匹配子规则：获取一些字符串，直到发现匹配后续子规则的字符为止

```
STRING : '"' .*? '"' ;  //匹配分隔符之间的任意文本
```

#### 匹配注释
```g4
LINE_COMMENT : '//' .*? '\n'  -> skip 
COMMENT :  '/*' .*? '*/'      -> skip
```


### 划定界线

#### 定义
* 词法分析器和语法分析器之间的界线

#### 经验法则
* 词法分析可以丢弃任何语法分析无须知道的东西
* 由词法分析来匹配词法符号（标识符、关键字、字符串、数字）
* 语法分析无须区分的词法结构可以归为同一个类型
* 词法分析可以将任何语法分析以相同方式处理的实体归为一类
* 如果语法分析需要把文本拆开处理，则词法分析就需要将其分成独立的词法符号



### 真实例子

#### CSV语法
```g4
grammar CSV;

file : hdr row+ ;
hdr : row ;
row : field (',' field)* '\r'? '\n' ;
field 
	: TEXT 
	| STRING 
	|         //指什么都没有
	;

TEXT : -[,\n\r"]+ ;  //下一个逗号或者换行符之前的任意字符序列
STRING : '"' ('""'|-'"')*  '"'  ;
```

#### JSON语法
```g4
grammar JSON

json : object | array ;

object 
	: '{' pair (',' pair)*   '}' 
	| '{' '}'  //空对象
	;
pair : STRING ':' value ;   //单独定义键值对
array 
    : '[' value (',' value)* ']'
    | '[' ']'
    ;
value
	: STRING
	| NUMBER
	| object
	| array
	| 'true'
	| 'false'
	| 'null'
	;

NUMBER
	: '-'? INT   //-3, 45
	| '-'? INT EXP  // 1e10, -3e4
	| '-'? INT '.' INT EXP?  //1.35 0.3 -4.5
	;
fragment INT : '0' | [1-9] [0-9]* ;
fragment EXP : [Ee] [+\-]? INT ;    // \- 是对-的转义
```


## 解耦

### 定义
* **使用监听器机制和访问器机制，来将语法和程序的逻辑代码解耦**

### 监听器

#### 特点
* 不需要显示的调用子节点的访问方法

#### 缺点
* 无法控制遍历过程


#### 流程
1. 语法分析器建立一颗语法分析树
2. 遍历该树，在过程中触发应用的逻辑代码

```java
ParseTreeWalker walker = new ParseTreeWalker();
PropertyFileLoader loader = new PropertyFileLoader();  //监听器对象
walker.walk(loader, tree)  //遍历语法分析树
```

#### 主要类
* ParseTreeListener  接口类，ANTLR运行库
* PropertyFileListener  接口类，由ANTLR自动生成
* ProperyFileBaseListener  默认实现类，由ANTLR自动生成
* PropertyFileLoader   应用类，实现主逻辑



### 访问器

#### 流程
* 使用  -vistor 选项生成访问者类
* 生成访问者对象，变量语法树

```java
PropertyFileVisitor loader = new PropertyFileVisitor()
loader.visit(tree)  //直接遍历语法分析树
```

#### 主要类
* PropertyFileVisitor<T>  接口类，由ANTLR自动生成
* PropertyFileBaseVisitor<T> 默认实现类，由ANTLR自动生成
* PropertyFileVisitor      应用类，实现主逻辑



### 标记备选分支
* 使用 ‘#’ 运算符给任意规则的最外层备选分支设置标签
* ANTLR会给每个备选分支都生成一个单独的监听器方法

```g4
e : e MULT e     # Mult
  | e ADD e      # Add
  | INT          # Int
  ;
```

### 共享数据

#### 方法
* 原生java调用栈
* 自定义栈
* 标注（首选）


#### 标注
* 优点：存储任意信息
* 缺点：局部变量会被保留，增大内存消耗
* 方法：使用map来将任意值和节点一一对应

```java
public static class EvalutorWithProps extends LExprBaseListener {
	//等于 map<ParseTree, Integer> 
	ParseTreeProperty<Integer> values = new ParseTreeProperty<Integer>();

    void setValue(ParseTree node, int value) { values.put(node, value) }
    int getValue(ParseTree node) { return values.get(node) }

    public void exitInt(LExprParser.IntCotext ctx) {
	    String intText = ctx.INT().getText();
	    setValue(ctx, Integer.valueOf(intText));
    }
		
	public void exitAdd(LExprParser.AddContext ctx) {
		int left = getValue(ctx.e(0)); // e + e
		int right = getValue(ctx.e(1));
		setValue(ctx, left + value)
	}
	
}
```

#### 原生java调用栈
* 优点：直接使用了访问器方法，具有良好的可读性
* 优点：空间效率比较高
* 缺点：无法使用自定义参数

#### 自定义栈
* 优点：传递多个参数值和多个返回值
* 优点：空间效率比较高
* 缺点：手工操作栈会有失误可能




## 语法参考

### 注释
* 单行  //
* 多行  /*  ...  */
* javadoc   /**   ....  */


### 标识符
* 词法符号名和词法规则名需要以大写字母开头
* 文法规则名以小写字母开头


### 文本常量
* 不区分 字符常量 和 字符串常量
* 由 单引号 括起来的字符串
* 不支持 正则表达式
* 支持常见的转义序列： '\n'  '\t'   '\f'

### 关键字
* import, fragment, returns
* lexer, parser, grammar
* locals, throws, catch,  finally


### 语法结构

#### 组成
* 语法声明
* 若干条规则

```g4
grammar Name;
import ...
options { ... }
tokens  { ... }

<<rule1>>  
ruleName : <<alternative1>> |  ... | <<alternativeN>> ;
....
<<ruleN>>
```

#### 总体
* 文法规则必须以小写字母开头
* 词法规则必须以大写字母开头
* 不带前缀的语法声明是混合语法 （ grammar name ）
* 纯文法规则声明带parser前缀
* 纯词法规则声明带lexer前缀

#### 语法导入
* 使用 import 来导入语法
* 导入会继承所有的规则、词法符号和具名动作
* 主语法规则会覆盖其导入语法中的规则
* 被导入的规则会放置在主语法的词法规则的末尾


### 文法规则

* 基本形式：规则名，备选分支， 分号
```g4
retstat : 'return' exprt ';'  ;
```

* 多个备选分支
```g4
stat : retstat 
     | 'break' ';'
     | 'continue'  ';'
     ;
```
     
* 备选分支可以包括一个空规则：使得整条规则成为可选的
```g4
superClass
	: 'extends' ID
	|
	;
```

#### 分支标签
* 使用#可以给备选分支添加标签
* 每个标签会生成对应的监听器回调函数和规则上下文类

```g4
e : e '*' e  # Mult
  | e '+' e  # Add
  | INT      # Int
  ;
```

#### 规则元素标签
* 使用 ’=‘ 符号来给规则元素增加标签
* 每个标签会成为规则上下文对象的字段
* 使用 ’+=‘ 符号可以增加一个列表字段

```g4
//value会生成一个普通对象字段
stat ； ’return' value=e ';'  # Return
     | 'break' ';'           # Break
     ;

//el会生成一个列表对象字段
array : '{'  el+=INT (',' el+=INT)*  '}'   ;
```

#### 子规则
* 使用括号包含的规则
* 子规则后加 **?** 表示可选，可以不匹配任何文本
* 子规则后加 **\*** 表示可以匹配零次或者多次
* 子规则后加 **+** 表示可以匹配一次或者多次


#### 文件结束符
* EOF 是预定义的词法符号 
* 表示必须要匹配所有的输入文本


### 词法规则

#### 概述
* **定义方式与文法规则类似**
* 不要在多条词法规则的右侧指定相同的字符串常量（存在歧义）

#### fragment
* 定义一些辅助规则，不会生成语法分析器可见的词法符号

```g4
INT : DIGIT+ ;
fragment DIGIT : [0-9] ;
```

#### 词法模式
* 用于将词法规则按上下文进行分组
* 使用mode指令来切换上下文

```g4

<<rules in default mode>>

mode MODEL1:
<<rules in MODE1>>


mode MODELN:
<<rules in MODELN>>
```

#### 指令
* 使用 -> 符号指定
* 可以使用多条指令，使用逗号分隔
* 指令可以携带参数

##### skip
* 对应的词法符号不会返回给语法分析器
```g4
WS : [ \r\t\n]+ -> skip ;
```

##### more
* 使用匹配的文件继续进行词法符号匹配
* 下一条词法符号规则的文本会包含有当前规则匹配的文本
```g4
//匹配字符串常量

LQUOTE : '"' -> more, mode(STR) ;
WS : [ \r\t\n]+ -> skip ;

mode STR;

STRING : '"' -> mode(DEFAULT_MODE);
TEXT : . -> more ;   //收集更多的文本，来生成字符串
```

##### type
* 为当前词法符号设置类型
```g4
tokens { STRING } ;

DOUBLE :  '"' .*? '"'  -> type(STRING) ;
SINGLE :  '\'' .*? '\''  -> type(STRING) ;
```

##### channel
* 为当前词法符号设置通道
* 默认值为 Token.DEFAULT_CHANNEL，整数值为0
* 隐藏通道为 Token.HIDDEN_CHANNEL, 整数值为1


##### mode
* 匹配到当前词法符号后，切换到指定模式



### 非贪婪规则

* 子规则  (,,,)?  (...)*  (...)+  都是贪婪的
* 贪婪规则会消费尽可能多的输入文本
* 在贪婪规则后增加 ? 后缀会使规则变成非贪婪的
* 非贪婪规则在词法分析器和语法分析器中都可以使用，词法分析中用的比较多
* 应当尽量少用非贪婪规则，会增加识别的复杂度

#### 词法规则

```g4
//匹配注释，.*?会匹配任意字符，直到遇见第一个 */ 为止
COMMENT : '/*' .*?  '*/'   -> skip ;   

//可以匹配转义双引号
STRING : '"'  ('\\"' | .)*?  '"'  ;   
```


#### 选择规则
词法分析器选择词法符号的规则
* 首先是选择能够识别最多输入字符的词法规则
* 如果有多个规则匹配相同的输入序列，则位置靠前的优先级更高
* 非贪婪规则匹配能够满足该规则的最少的字符序列
* 在非贪婪规则后的所有决策都遵循  **首先匹配** 原则
