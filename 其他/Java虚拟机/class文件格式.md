

# class文件格式

## 概述

1. java程序二进制格式的精确定义 
2. 8位字节的二进制流 
3. 按大端格式存放多字节数据

## 文件格式

### 组成部分
* version_info 版本号信息 
* cp_info[] 常量池 
* flag_info / class 其他信息 
* interface_info[] 接口类信息 
* field_info[] 字段信息 
* method_info[] 方法信息 
* attribute_info[] 属性信息

#### 注意项
* 常量池是变长的，但使用常量池是使用索引号的
* 使用引用项来完成符号链接

## 常量池

### 概述
* 一个包含可变长度cp_info项的数组 
* 项包含 tag（一个字节） + 内容（变长） 
* 项类型： 内容项、引用项（符号引用） 
* 用于保存class文件中的所有字符串数据

### 内容项
* 保存实际数据 
* CONSTANT_utf8  utf8编码的字符串 
* CONSTANT_Integer int类型的值 
* CONSTANT_Float float类型的值 
* CONSTANT_Long long类型的值 
* CONSTANT_Double double类型的值

### 引用项
* 指向其他项 
* CONSTANT_String string类型的值 
* CONSTANT_NameAndType 描述符号引用 
    * CONSTANT_Class 对类、接口的符号引用 
    * CONSTANT_Fieldref 对字段的符号引用 
    * CONSTANT_Methodref 对类中方法的符号引用 
    * CONSTANT_InterfaceMethodref 对接口中方法的符号引用


 ## 符号引用

 ### 字符串类型
 * 全限定名： 类名、接口名 ---- java/lang/Object 
 * 简单名字：方法名、字段名 ---- toString 
 * 描述符： 描述字段、方法 ---- 由上下文无关语法定义   


 ### 描述符语法
 
 #### 基本类型
 * B --- byte
 * C --- char
 * Z --- boolean
 * I ---- int 
 * D --- double
 * F --- float
 * J --- long 
 
 #### 其他
 * [ ---- 数组
 * L; ---- 类
 * () ---- 函数 
 
 #### 字段描述符
 * [[J ----- long[][] road
 * [Ljava/lang/Object; ----- java.lang.Object[] stuff

 #### 方法描述符
 * ()I ---- int getSize(); 
 * ([BII)I ---- int read(byte[] b, int off, int len);


 ### 例子
 
 ####  CONSTANT_Methodref项 
 * tag ---- 10 
 * class_index ---- CONSTANT_Class项索引 
 * name_and_type_index ---- CONSTANT_NameAndType项索引 
 
 #### CONSTANT_Class项 
 * tag ----- 7 
 * name_index ----- CONSTANT_utf8项索引 (类/接口的全限定名) 
 
 #### CONSTANT_NameAndType项 
 * tag ----- 12 
 * name_index ----- CONSTANT_utf8项索引 (字段/方法的名字) 
 * descriptor_index ---- CONSTANT_utf8项索引 (字段/方法的描述符)


## 描述信息

### 字段信息

#### 功能
* 描述类/接口中声明的每一个字段 

#### 属性
* access_flags ----- 修饰符信息 
* name_index ---- 名字常量池索引 
* descriptor_index ----- 描述信息常量池索引 
* attributes_info ---- 属性信息 
    * ConstantValue


### 方法信息

#### 功能
* 描述类/接口中声明的,由编译器产生的每一个方法 

#### 属性
* access_flags ---- 修饰符信息 
* name_index ---- 名字常量池索引 
* descriptor_index ----- 描述信息常量池索引 
* attributes_info ---- 属性信息 
    * Code 
    * Exceptions

### 属性信息

#### Code属性
##### 功能
* 描述字节码信息

##### 属性
* attribute_name_index ---- 属性名字索引(字符串"Code") 
* attribute_length ---- 属性长度 
* max_stack ----- 栈的最大长度 
* max_locals ----- 局部变量的最大长度 
* code_length, code ----- 字节流

* exception_table_length ---- 异常表项个数 
* exception_table ----- 异常表项
    * start_pc , end_pc ---- 异常处理器的范围(指令偏移量) 
    * handler_pc ----- 异常处理器的指令偏移量 
    * catch_type ---- 异常类型的索引

* attributes_count ----- code属性个数
* attributes_info ---- code 属性信息 
    * LineNumberTable 
    * LocalVariableTable

#### ConstantValue属性 
##### 功能
* 描述常量值信息 

##### 属性
* attribute_name_index ---- 属性名字索引(字符串"ConstantValue") 
* attribute_length ---- 属性长度 
* constantvalue_index ---- 常量值的常量池索引