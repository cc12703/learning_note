

# Smali格式


## 数据类型
* V    void 只能用于返回值类型
* Z    boolean
* B    byte
* S    short
* C    char
* I      int
* J      long
* F     float
* D    double
* L     java类类型
    * 例子： Ljava/lang/String;
* [      数组类型
    * 例子： [I  或  [[Ljava/lang/String;


## 寄存器表示
* 使用Dalvik的p命名法
* 函数的入参从p0开始命名
* 局部变量从v0开始命名
* 非静态方法，p0表示隐式对象调用，p1才是第一个参数


## 文件结构
### 文件头
> .class <访问权限>  [修饰关键字]  <类名>
> .super <父类名>
> .source <源文件名>


### 方法
> .method <访问权限> [修饰关键字] <方法原型>
> .locals <局部变量个数>
> .param <方法的参数>
> .prologue    //代码开始
>   代码体
> .end method


### 成员变量
#### 静态
> .field <访问权限> [修饰关键字] <字段名>:<字段类型>

```
# static fields
.field private static count:I
```

#### 动态
> .field <访问权限> [修饰关键字] <字段名>:<字段类型>

```
# instance fields
.field private name:Ljava/lang/String;
```


### 标注
> .annotation [注解属性] <注解类名>
> [注解字段 = 值] 