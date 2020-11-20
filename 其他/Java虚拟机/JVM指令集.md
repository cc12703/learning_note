

# JVM指令集

## 概述

* 操作对象： 栈帧中的操作数栈 
* 输入：操作数栈、自带的参数 
* 输出：操作数栈 


## 分类

### 栈操作
#### 常量入栈
* aconst_null 将NULL对象引用压入栈 
* Xconst_Y  将X类型常量Y压入栈 
* ldc  将常量池中的项压入栈 
* ldc_w 将常量池中的项压入栈(使用宽索引) 
* bipush  将8位带符号整数压入栈
* sipush  将16位带符号整数压入栈

#### 局部变量入栈
* Xload_Y 从局部变量Y中，将X类型值压入栈 

#### 局部变量出栈 
* Xstore_Y  将X类型值出栈，存入局部变量Y中 

#### 通用操作 
* pop  弹出栈顶的一个字 
* pop2 弹出栈顶的两个字 
* swap  交换栈顶部的两个字 
* dup  复制栈顶部的一个字 
* dup2 复制栈顶部的两个字 
* dup_x1

### 对象操作
#### 单一对象
* new 创建一个新对象,将引用压入栈 
* getfield 从对象中获取字段 
* putfield 设置对象中字段的值 
* getstatic 从对象中获取静态字段 
* putstatic 设置对象中的静态字段 
* instanceof 判断对象是否为给定的类型 

#### 数组对象
* newarray  创建新的基本数据类型数组 
* anewarray  创建新的应用类型数组 
* multianewarray  创建新的多维数组 
* arraylength  获取数组长度 
* Xaload  从数组中，将X类型值压入栈 
* Xastore 将X类型值出栈，存入数组中

### 方法调用
* invokevirtual  调用实例方法，使用动态绑定(根据运行时的对象) 
* invokespecial 调用实例方法，使用静态绑定(根据编译时类型) 
* invokestatic  调用类方法，使用静态绑定 
* invokeinterface 调用接口方法，使用动态绑定 
* Xreturn 从方法中返回特定类型的数据

### 运算
* X2Y  将X类型数据转换成Y类型 
* Xadd 执行X类型的加法 
* Xsub 执行X类型的减法 
* Xmul 执行X类型的乘法 
* Xdiv  执行X类型的除法 
* Xrem  计算X类型的余数 
* Xneg  对X类型数据进行取反操作 
* Xshl  将X类型数据左移位 
* Xshr 将X类型数据右移位 
* Xushr  将X类型数据逻辑右移位 
* Xand  执行X类型的逻辑与 
* Xor  执行X类型的逻辑或 
* Xxor  执行X类型的逻辑异或


### 控制流
* goto 无条件跳转 
* goto_w  无条件跳转(使用宽索引) 
* tableswitch  通过索引值访问跳转表，进行跳转 
* lookupswitch  通过键值匹配访问跳转表，进行跳转 
* athrow  抛出异常
* finally子句：
    * jsr 跳转到子例程 
    * jsr_w  跳转到子例程(使用宽索引) 
    * ret 从子例程返回
* ifZ  如果条件Z成立，则跳转 
* if_icmpZ  比较两个int类型值，若条件Z成立，则跳转 
* ifnull  若等于null, 则跳转 
* ifnonnull  若不等于null, 则跳转 
* Xcmp  比较X类型值

### 其他
* monitorenter  获取对象监视器 
* monitorexit  释放对象监视器 
* wide  扩展局部变量索引(从8位到16位) 
* checkcast  确定对象是否为给定的类型 
* instanceof  判断对象是否为给定类的实例

### quick指令
**一种加速字节码解释的技术，使用这些指令来表示常量池已解析过了**
* ldc_quick 
* getfield_quick 
* putfield_quick 
* lgetstatic_quick 
* putstatic_quick 
* new_quick 
* invokevirtual_quick 
* invokestatic_quick


## 描述

### 一般指令

#### istore
* 功能：将int类型值存入局部变量中 
* 格式：istore, index （当前栈帧内局部变量的索引值） 
* 栈参数： ...., value （要保存的数据） 
* 过程：从栈中弹出数据，将其放入当前栈帧的局部变量区域中 

#### new
* 功能： 创建一个新对象 
* 格式： new , indexbyte1 , indexbyte2 (常量池的索引) 
* 栈结果： ..., objectref （对象引用） 
* 过程：查找常量池入口，解析该入口，从堆中分配对象，将对象引用压入栈 

#### newarray 
* 功能: 创建一个数组对象，成员为基本数据类型 
* 格式： newarray , atype (数据类型) 
* 栈参数： count (数组大小) 
* 栈结果： arrayref (数组对象引用) 

#### iadd 
* 功能： 执行int类型的加法 
* 格式：iadd 栈参数：....., value1, value2 
* 栈结果：....., result 
* 过程：从栈中弹出要相加的数据，执行加法操作，将结果压入栈


### 方法调用

#### invokevirtual 
* 功能： 调用实例方法 使用动态绑定，根据对象实际的类来调用方法 
* 格式： invokevirtual , indexbyte1 , indexbyte2 (常量池的索引) 
* 栈参数： objectref, arg1, arg2, .... 
* 过程：
    * 解析常量池人口（CONSTANT_Methodref_info）获取方法名字、描述信息 
    * 在对象对应的类中搜索方法（搜索父类） 
    * 从栈中弹出参数，并放入新栈的局部变量中 
    * 执行要调用方法的字节码


 #### invokestatic
 * 功能： 调用静态方法 使用静态绑定，根据引用的类来调用方法 
 * 格式： invokestatic, indexbyte1 , indexbyte2 (常量池的索引) 
 * 栈参数： arg1, arg2, .... 
 * 过程：
    1. 解析常量池入口（CONSTANT_Methodref_info)，获取引用类、方法名、描述信息 
    2. 在引用类中搜索方法 从栈中弹出参数，并放入新栈的局部变量中
    3. 执行要调用方法的字节码


#### invokeinterface 
* 功能： 调用接口方法 使用动态绑定，但是不能假定同一方法在不同类的方法表中有相同的偏移量 
* 格式： invokeinterface, indexbyte1 , indexbyte2 (常量池的索引) 
* 栈参数： objectref , arg1, arg2, .... 
* 注意：
    * 常量池入口为 CONSTANT_InterfaceMethodref_info 
    * 每次都必须搜索方法表，不能假定偏移量相同


#### invokespecial 
* 功能：调用实例方法 使用静态绑定，除了一个例外 
* 格式： invokespecial, indexbyte1 , indexbyte2 (常量池的索引) 
* 栈参数： arg1, arg2, ....
* 适用情况
    * 实例初始化方法 init() 
    * 类的私有方法 
    * 使用super关键字所调用的方法
* 例外情况
    * class类的access_flags设置了ACC_SUPER 
    * 解析指向超类方法的符号引用 
    * 处理：使用动态绑定，在当前类的超类中搜索该方法


### 其他指令

#### checkcast 
* 功能：确定对象是否为给定的类型 
* 格式：checkcast, indexbyte1 , indexbyte2 (常量池的索引) 
* 栈参数： ..., objectRef (对象引用) 
* 算法：给定的类型是java.lang.object 
    * 对象是类实例： 
        * 给定的类型是类，对象所属的类是该类或其子类 
        * 给定的类型是接口，对象必须实现该接口 
    * 对象是数组： 
        * 给定的类型必须和对象是同一种数组 
        * 若是基本数据类型，则必须是一样的基本数据类型
        * 若是引用类型，则必须是同样的引用类型


