

# Jenkins-JCasC


## 功能
* 使用配置文件来配置Jenkins


## 实现
* 配置文件使用yaml格式
* 每一个节点都会被传递给一个Configurator对象
* 该对象将配置信息应用到jenkins对象上


### Configurator
* 用于管理一个特定的Jenkins组件

#### 组成
* name : 用于匹配对应的yaml条目
* target : 对应的组件类型
* describe函数 ： 导入组件的属性信息用于配置
* configure函数 ： 配置组件

### 根Configurator
* 使用特定的RootElementConfigurator接口类，来管理yaml文件中的根元素
* JCasC提供了一个自定义的对象来管理jenkins根元素

### 属性Attributes
* describe函数会返回一个attribute的集合
* 一个attribute对应一个setter方法
* JCasC会使用反射来解析出名字和参数类型
* 被标记为Deprecated和Restricted的方法会被排除

### 通用机制

#### 数据绑定
* 该机制是用于配置web界面的方法
* 使用DataBoundConstructor标注Jenkins组件的构造函数
* 使用DataBoundSetter标注setter方法
* 使用Symbol标准来修改名字


## 插件

