

# SpringBoot

[TOC]

## 总体

### 概述
* SpringBoot是一个机遇Spring框架搭建起来的应用
* SpringBoot为了让Spring框架更容易得到快速的使用
* SpringBoot致力于快速应用开发领域



### 优点
* 内嵌了Tomcat、Jetty等服务器，不需要war文件进行部署
* 通过Maven来根据需要获取starter
    * starter可以直接获取开发所需的相关包
* 尽可能地自动配置Spring
    * 配置尽量简单且存在约定
    * 约定优于配置，是其主导思想
* 使得Spring经过简单配置后即可进行使用开发



## IoC

### 概述
* Ioc 控制反转，是Spring的核心
* IoC 是一种通过描述来生成、获取对象的技术
* Spring Boot不建议使用XML，而是通过注解的描述生成对象

### 概念
#### Bean
* 需要管理的对象
* 全称为Spring Bean

#### Ioc容器
* 一个管理Bean的容器
* 包括两个功能
    * 通过描述管理Bean，包括发布和获取
    * 通过描述完成Bean之间的依赖关系


### Bean生命周期
1. Bean的定义
1. Bean的初始化
1. Bean的生存期
1. Bean的销毁



## AOP

### 概述
* AOP的本质是约定
* 通过与开发者的约定，把对应方法通过动态代理技术织入到约定的流程中
* Spring Boot可以使用@AspectJ注解启用AOP


### 约定编程
#### 简易接口
```java
public interface HelloService {
    public void say Hello(String name);
}
```

#### 拦截器接口
```java
public interface Interceptor {

    public boolean beofre(); //事前方法
    public void after(); //事后方法

    public Object around(); //取代原有方法

    public void afterReturning(); //事件没有发生异常时执行
    public void afterThrowing(); //事件发生异常后执行

}
```

#### 约定
当调用proxy对象的方法时，执行流程如下
1. 先执行拦截器的before()
1. 调用target对象的事件方法
1. 执行烂机器的after()
1. 如果发生异常，则执行拦截器的afterThrowing()，否则调用afterReturning()

#### ProxyBean
* 使用动态代理技术，将简易接口和拦截器方法织入约定的流程中
* getProxyBean()会返回一个代理对象


### 概念
* 连接点（join point）：具体被拦截的对象，Spring中特定的方法
* 切点（point cut）：使用正则表达式和指示器的规则去适配多个连接点
* 通知（advice）：约定流程中的方法
    * 前置通知（before）、后置通知（after）
    * 环绕通知（around）
    * 事后返回通知（afterReturning）、异常通知（afterThrowing）
* 目标对象（target）：被代理的对象
* 引入（introduction）：引入新的类和其方法，增强现有Bean的功能
* 织入（weaving）：指以下操作流程
    1. 通过动态代理技术，为原有对象生成代理对象
    1. 拦截与切点定义匹配的连接点
    1. 按约定将各类通知织入约定流程
* 切面（aspect）：一个可以定义切点、各类通知和引入的内容


### 步骤
#### 确定连接点
* Spring中什么类的什么方法需要使用AOP

#### 开发切面
* 使用@Aspect来声明切面
* 使用@Before,@After,@AfterReturning,@AfterThrowing来定义通知
* 正则表达式用于定义什么时候启用AOP
    * Spring会通过正则表达式去匹配、去确定对于的方法是否启动切面编程

```java
@Aspect
public class MyAspect {

    @Before("execution(* com.xxx.UserServiceImpl.printUser(...))")
    public void before() {
        System.out.println("before .....")
    }
}
```
说明
* execution 在执行方法时，拦截正则表达式匹配的方法
* \* 返回值为任意类型
* com.xxx.UserServiceImpl 目标对象的全限定名称
* printUser 目标对象的方法
* (...) 匹配任意参数

#### 切点定义
* 用于向Spring描述哪些类的哪些方法需要启用AOP

```java
@Aspect
public class MyAspect {

    @Pointcut("execution(* com.xxx.UserServiceImpl.printUser(...))")
    public void pointCut() { }

    @Before("pointCut()")
    public void before() {
        System.out.println("before .....")
    }

    @After("pointCut()")
    public void after() {
        System.out.println("after .....")
    }
}
```



## 访问数据库

### 技术
* JdbcTemplate + JDBC
* JPA + Hibernate
* MyBatis


### JPA+Hibernate
* JPA (Java持久化API)：定义了ORM以及实体对象持久化的标准接口
* Hibernate：一个全映射框架，重模型和业务分析

#### JPA核心
* 核心：实体 + 持久化上下文
* 持久化上下文：ORM(对象关系映射)，实体操作API，查询语言(JPQL)


### MyBatis
* 一个持久层框架，支持定制化SQL
* 将接口和POJO映射成数据库中的记录

#### 配置
* 基础配置
* 映射文件