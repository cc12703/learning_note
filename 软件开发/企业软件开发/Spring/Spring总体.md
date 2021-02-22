

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



## 配置属性

### 概述
* 配置属性是Spring应用上下文中bean的属性
* 可以通过多个源进行设置

#### 环境抽象
* Spring本身抽取了原始的属性，需要属性的bean可以从Spring中获取
* 属性源
    * JVM系统属性
    * 操作系统的环境变量
    * 命令行参数
    * 应用属性配置文件
        * application.properties
        * application.yml

##### 图示
![](https://gitee.com/cc12703/figurebed/raw/master/img/20210223182548.png)


### 配置示例
#### 数据源
```yml
spring:
    datasource:
        url: jdbc:mysql://localhost/tacocloud 
        username: tacodb
        password: tacopass 
        driver-class-name: com.mysql.jdbc.Driver 

        schema:  #创建表结构
        - order-schema.sql
        - user-schema.sql

        data:   #填充数据
        - ingredient.sql
```

#### 嵌入式服务器
```yml
server:
    port: 8443
    ssl: 
        key-store: file://path/mykeys.jks
        key-store-password: letmein
        key-password: letmein
```

#### 日志
```yml
loggin:
    path: /var/logs/
    file: TacoCloud.log
    level:  # 设置日志级别
        root: WARN
        org.springframework.security: DEBUG
```


### 自定义属性
* 使用@ConfigurationProperties标注
* 通过创建持有者bean，将所有属性收集在一个地方
* 通过META-INF添加元数据
    * src/main/resources/META-INF目录
    * additional-spring-configuration-metadata.json文件

#### 定义
```kotlin
@Comonent
@ConfigurationProperties(prefix="taco.orders")
@Data
public class OrderProps {

    @Min(value=5, message="")
    @Max(value=10, message="")
    private int pageSize = 20;
}
```

#### 使用
```yml
taco:
    orders: 
        pageSize: 10
```


### profile
* 一种条件化的配置
* 在运行时，根据哪些profile处于激活状态，可以使用或忽略不同的属性

#### 定义
* 使用独立的配置文件
    * 命名规则：application-{profile_name}.yml
* 使用一个配置文件，进行分段
    ```yml
    # 通用配置
    logging:
        level:
            tacos: DEBUG
    
    # 条件配置
    --- 
    spring: 
        profiles: prod
    
    logging:
        level:
            tacos: WARN
    ```

#### 激活
* 在application.yml中设置spring.profiles.active属性（不推荐）
* 使用环境变量SPRING_PROFILES_ACTIVE（推荐）


#### 按profile创建bean
* 某些bean只在特定profile激活的情况下才需要创建
* 使用@Profile标注
```java

@Bean
@Profile("dev")
public CommandLineRunner dataLoader(....) {
    ...    
}
```



## 保护Spring

### 概述
* 需要增加依赖spring-boot-starter-security 
* 默认安全特性
    * 所有HTTP请求路径都需要认证
    * 没有特定的角色和权限
    * 没有登录界面
    * 使用HTTP basic进行认证
    * 系统只有一个用户，名字user
* 配置通过继承WebSecurityConfigurerAdapter进行
    ```java
    @Configuration
    @EnableWebSecurity
    public class SecurityConfig extends WebSecurityConfigurerAdapter {
        ...
    }
    ```

### 用户存储
#### 配置
* 通过覆盖configure(AuthenticationManagerBuilder auth)进行

#### 方案
* 基于内存的
* 基于JDBC的
* 基于LDAP后端的
* 自定义的


### 保护Web请求
#### 配置
* 通过覆盖configure(HttpSecurity htpp)进行

#### 功能
* 为请求提供服务前，需要预先满足特点的条件
    ```java
    .authorizeRequests()
        .antMatchers("/design", "/orders").hasRole("ROLE_USER")
        .antMatchers("/", "/**").permitAll()
    ```
* 配置自定义的登录界面
    ```java
    .and().formLogin() 
          .loginPage("/login") //设置登录URL
          .defaultSuccessUrl("/design")  //设置登录成功后的跳转URL
    ```
* 支持用户退出应用
* 预防跨站请求伪造


### 了解用户是谁
* 通过@AuthenticationPrincipal注入
```java
@PostMapping
public String processOrder(@Valid Order order,
                    Errors errors, 
                    @AuthenticationPrincipal User user)
```



## 集成Spring



## 监控Spring

### Spring Boot Actuator
* 通过暴露端点，可以提供应用的运行状态信息
* 依赖：spring-boot-starter-actuator

#### 端点
| 路径  | 描述 |
| --- | --- |
| /health | 获取健康状态 |
| /configprops | 获取配置属性及其值 |
| /info    | 获取应用信息   |
| /metrics | 获取指标分类列表 |

#### 配置端点
* 配置属性 management.endpoints.web.exposure.include
* 配置属性 management.endpoints.web.exposure.exclude


### Spring Boot Admin
* 一个管理类的Web前端应用
* 获取actuator提供的信息，并进行展示

#### 结构
* 服务器：收集数据、展示数据
* 客户端：提供数据、一个/多个Spring Boot应用

#### 创建服务器
* 作为一个独立应用
* 创建一个新的Spring Boot项目，并加入admin依赖

#### 注册客户端
* 每个应用显示向服务器注册
    1. 在应用中加入spring-boot-admin-starter-client依赖
    2. 将配置spring.boot.admin.client.url设置为admin服务器
* 服务器通过**注册中心**发现客户端