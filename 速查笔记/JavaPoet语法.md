
# JavaPoet语法


**JavaPoet是一套用于生成 .java源文件的API**

## 例子

目标文件 HelloWorld.java
```java
package com.example.helloworld;

public final class HelloWorld {
  public static void main(String[] args) {
    System.out.println("Hello, JavaPoet!");
  }
}
```	

生成代码
```java
MethodSpec main = MethodSpec.methodBuilder("main")
    .addModifiers(Modifier.PUBLIC, Modifier.STATIC)
    .returns(void.class)
    .addParameter(String[].class, "args")
    .addStatement("$T.out.println($S)", System.class, "Hello, JavaPoet!")
    .build();

TypeSpec helloWorld = TypeSpec.classBuilder("HelloWorld")
    .addModifiers(Modifier.PUBLIC, Modifier.FINAL)
    .addMethod(main)
    .build();

JavaFile javaFile = JavaFile.builder("com.example.helloworld", helloWorld)
    .build();

javaFile.writeTo(System.out);
```

## 模型

* TypeSpec   表示 类 和 接口
* FieldSpec  表示 字段
* MethodSpec 表示 方法 和 构造函数
* ParameterSpec 表示 参数
* AnnotationSpec 表示 标注


## 替换符

### \$L   
* 替换为常量 
	  
### \$S  
* 替换为字符串常量（带双引号）


### \$T  

* 引用数据类型，自动生成import语法
```java
MethodSpec today = MethodSpec.methodBuilder("today")
    .returns(Date.class)
    .addStatement("return new $T()", Date.class)
    .build();

TypeSpec helloWorld = TypeSpec.classBuilder("HelloWorld")
    .addModifiers(Modifier.PUBLIC, Modifier.FINAL)
    .addMethod(today)
    .build();

JavaFile javaFile = JavaFile.builder("com.example.helloworld", helloWorld)
    .build();

javaFile.writeTo(System.out);
```
 
### \$N  
* 使用名字来引用另一个已生成的声明
```java
MethodSpec hexDigit = MethodSpec.methodBuilder("hexDigit")
    .addParameter(int.class, "i")
    .returns(char.class)
    .addStatement("return (char) (i < 10 ? i + '0' : i - 10 + 'a')")
    .build();

MethodSpec byteToHex = MethodSpec.methodBuilder("byteToHex")
    .addParameter(int.class, "b")
    .returns(String.class)
    .addStatement("char[] result = new char[2]")
    .addStatement("result[0] = $N((b >>> 4) & 0xf)", hexDigit)
    .addStatement("result[1] = $N(b & 0xf)", hexDigit)
    .addStatement("return new String(result)")
    .build();
```

## 构造方法
* 使用MethodSpec.constructorBuilder()生成


## 方法参数
* 使用ParameterSpec.builder() 或者 MethodSpec.addParameter() 生成

```java
ParameterSpec android = ParameterSpec.builder(String.class, "android")
              .addModifiers(Modifier.FINAL) .build(); 

MethodSpec welcomeOverlords = MethodSpec.methodBuilder("welcomeOverlords")
			  .addParameter(android) 
			  .addParameter(String.class, "robot", Modifier.FINAL) 
			  .build();
```

## 字段
* 使用FieldSpec.builder() 或者 TypeSpec.addField() 生成
* 当字段中包含javadoc, 标注，初始化语句时需要使用builder的方式

```java
FieldSpec android = FieldSpec.builder(String.class, "android")
            .addModifiers(Modifier.PRIVATE, Modifier.FINAL)
            .initializer("$S + $L", "Lollipop v.", 5.0d)
            .build();
```
