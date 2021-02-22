

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