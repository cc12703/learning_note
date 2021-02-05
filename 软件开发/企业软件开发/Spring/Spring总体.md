

# Spring总体

[TOC]

## 概述

### 组件
#### Spring核心框架
* 提供核心容器和依赖注入的功能
* 提供Spring MVC （Spring的Web框架）
* 对数据持久化的基础支持：JdbcTemplate

#### Spring Boot
* 提供starter依赖
* 提供自动配置
* Actuator：查看应用运行时的内部工作状况 
* 提供环境属性规范
* 命令行接口

##### DevTools
* 提供代码变更后，应用自动重启的功能
* 提供资源变更后，自动刷新浏览器的功能
* 提供自动禁用模板缓存的功能
* 内置H2控制台

#### Spring Data 
* 将数据repository定义为简单的Java接口
* 处理不同类型的数据库

#### Spring Security
* 安全框架，解决通用的安全性需求
* 包括：身份验证、授权、API安全性

#### Spring Integration
* 解决了实时集成问题
* 数据在可用时马上就会得到处理

#### Spring Batch
* 解决了批处理集成问题
* 数据会收集一段时间，直到某个触发器发出信号，数据才会被批量处理




## 项目结构

### 文件目录
* src/main/java/ 应用源码
* src/test/java/ 测试代码
* src/main/resources/ 非Java资源
    * static/  存放浏览器的静态内容
    * templates/ 存放模板文件
    * application.properties 定义配置属性
* mvnw和mvnw.cmd Maven包装器
* pom.xml Maven构建配置
* XXXApplication.java  Spring Boot主类，用于启动项目

### 关键文件
#### pom.xml
```xml
<project>
    <modelVersion>4.0.0</modelVersion>

    <groupId>sia</groupId>
    <artifactId>taco-cloud</artifactId>
    <version>1.0.0</version>
    <packaging>jar</packaging> <!-- 打包为JAR -->

    <name>taco-cloud</name>
    <description>....</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.4.RELEASE</version> <!-- Spring Boot的版本 -->
    </parent>

    <properties>
        ...
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

#### 引导类
```java
@SpringBootApplication
public class TacoCloudApplication {

    public static void main(String[] args) {
        SpringApplication.run(TacoCloudApplication.class, args);
    }
}
```
标注SpringBootApplication说明：包括了三个标注
* @SpringBootConfiguration：将类声明为配置类
* @EnableAutoConfiguration：启用Spring Boot的自动配置功能
* @ComponentScan：启用组件扫描，自动发现组件


#### 处理Web请求
```java
@Controller
public class HomeController {

    @GetMapping("/")
    public String home() {
        return "home";   // 返回视图名
    }
}
```



## 数据

### JdbcTemplate
* 用于支持使用JDBC来操作数据
* 避免使用JDBC时常见的样板式代码

#### 使用步骤
1. 标注数据对象
```java
@Data
public class Taco {
    private Long id;
    private Date createdAt;
}

@Data
public class Order {
    private Long id;
    private Date placedAt;
}
```
1. 增加依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-start-jdbc</artifactId>
</dependency>
```
1. 定义repository
```java
public interface IngredientRepository {
    Iterable<Ingredient> findAll();
    Ingredient findOne(String id);
    Ingredient save(Ingredient ingredient);
}
```
1. 实现repository
```java
@Repository  //用于自动发现
public class JdbcIngredientRepository 
            implements IngredientRepository {
    private JdbcTemplate jdbc;

    @Autowired  //用于依赖注入
    public JdbcIngredientRepository(JdbcTemplate jdbc) { 
        this.jdbc = jdbc;
    }

    public Iterable<Ingredient> findAll() {
        return jdbc.query("select id, name, type from Ingredient", 
                            this::mapRowToIngredient);
    }
}
```


### JPA 
* 基于repository规范接口自动生成repository

#### 使用步骤
1. 增加依赖
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-start-data-jpa</artifactId>
</dependency>
```
1. 标注数据对象
```java
@Data
@Entity
public class Taco {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;

    @NotNull
    @Size(min=5, message="xxxxxxx")
    private String name;

    private Date createdAt;

    @PrePersist
    void createdAt() {
        this.createdAt = new Date();
    }
}

@Data
@Entity
@Table(name="Taco_Order")
public class Order {
    @Id
    @GeneratedValue(strategy=GenerationType.AUTO)
    private Long id;

    private Date placedAt;

    @ManyToMany(targetEntity=Taco.class)
    private List<Taco> tacos = new ArrayList<Taco>();

    @PrePersist
    void placedAt() {
        this.placedAt = new Date();
    }
}
```
1. 声明repository
```java
public interface TacoRepository
        extends CrudRepository<Taco, Long> {
}

public interface OrderRepository
        extends CrudRepository<Order, Long> {
}
```





## 保护Spring

