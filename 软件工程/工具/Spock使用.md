
# Spock使用

[TOC]


## 概述
* Spock可以让你编写规格用于描述一个系统的被期望的特性（属性和方面）
* 这个系统可以是任何东西，从单独的一个类到整一个应用，这种方式也被称为系统根据规格(SUS)
* 一个特性的描述开始于一个具体的SUS快照和其配套
* 这个快照被称为特性的夹具(fixture)。

## 导入
* spock.lang包含了最重要的用于写规格的类型。

### 例子
```groovy
import spock.lang.*
```


## 规格
* 一个规格被表示为一个Groovy类，继承于spock.lang.Specification类
* 规格名字最好关联到规格所描述的系统或者系统操作上。例如，CustomerSpec, H264VideoPlayback
* 类Specification包含了许多用于写规格的有用的方法
* 使用Sputnik（Spock`s JUnit runner）可以在JUnit上运行规格

### 例子
```groovy
class MyFirstSpecification extends Specification {
  // fields
  // fixture methods
  // feature methods
  // helper methods
}
```


## 字段
* 实例字段可以用来存放属于这个规格的对象
* 保存在实例字段中的对象不会在特征方法间共享
* 每个特征方法都会获取到自己的对象（用于隔离各个特征方法）
* 在定义的地方初始化这些对象是一个最佳实践
* @Shared用于标注需要在特征方法间共享对象
* 静态字段只能用于常量。不然最好使用共享字段

### 例子
```groovy
def obj = new ClassUnderSpecification()
def coll = new Collaborator()

@Shared res = new VeryExpensiveResource()

static final PI = 3.141592654
```


## 夹具方法
* 夹具方法用于在特征方法运行前后，建立和清除环境
* setup()和cleanup()用来为每个特征方法使用一个更新后夹具
* setupSpec()和cleanupSpec()用于多个特征方法共享一个夹具

#### 例子
```groovy
def setup() {}          // run before every feature method
def cleanup() {}        // run after every feature method
def setupSpec() {}     // run before the first feature method
def cleanupSpec() {}   // run after the last feature method
```


## 特征方法
* 特征方法是规格的核心。用来描述一些特征（属性和特征）
* 为了方便，特征方法使用字符串进行命名

### 四个阶段
1. 建立特征夹具(可选的)
1. 提供符合系统规范的刺激行为（可以有多个）
1. 描述符合期望的系统响应 （可以有多个）
1. 清除特征夹具（可选的）

### 例子
```groovy
def "pushing an element on the stack"() {
	// blocks go here
}
```


### 方法块

|Phases| Blocks |
| -- | -- |
| Setup | setup: |
| Sitmulus  |  when:, expect: |
| Response  | then:, expect: |
| Cleanup   | cleanup: |



