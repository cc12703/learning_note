

# Gradle语法

## 基本操作

- Gradle中最基本的两个概念是 **project**, **task**  
- 每个构建由一个或多个project组成，project表示一件要完成的事情
- 每个project由一个或多个task组成，task表示一些原子性的工作

### 创建任务
```groovy
task hello {
    doLast {
       println ‘xxxx'
    }
}
```       

### 快捷创建
```groovy
task hello << {
     println  ‘xxx'
}
```

### 任务依赖
* 依赖的任务在添加时可以不存在

```groovy
task hello << {
	println 'xxx'
}

task intro(dependsOn: hello) {
	println 'xxx' 
}
```

### 操作已存在的任务
* << 等于 doLast 操作，doFirst, doLast可以被多次调用

```groovy
task hello << {
	println 'xxx'
}
hello.doFirst {
	println 'yyy'
}
hello.doLast {
	println 'zzz'
}
```

### 额外属性
* 使用ext.myProperty来设置初始值

```groovy
task myTask {
	ext.myProperty = 'myValue'
}
task printTaskProperty {
	println myTask.myProperty
}
```

### 默认任务
* 可以定义一个或者多个默认任务
* 在多project构建中，每个project都可以定义自己的默认任务


## 构建脚本

* Gradle提供了DSL用于描述构建，该语言基于Groovy，默认编码为UTF8

### Project API

- Gradle中，构建脚本为每个项目都定义了一个project对象，类型为Project
- 对于脚本中任何一个没有定义的方法，调用它都会被委托给project对象
- 对于脚本中任何一个没有定义的属性，访问它都会被委托给project对象

### Script API

- **定义变量**有两种类型的变量可以使用：本地变量、额外属性
- **本地变量**通过def关键字进行定义，只在被定义的区域内可见
- **额外属性**所有的Gradle领域模型对象都可以持有用户自定义的属性，使用ext进行设置



## 更多任务操作

### 定义任务
```groovy
task(hello) << {
	println 'xxx'
}
task('hello', type:Copy) {
	from('xxx')
	into('xxx')
}
tasks.create(name:'hello') << {
	println 'xxx'
}
```

### 定位任务

- 任务名可以作为project的属性名使用
- 使用tasks集合来定位
- 使用tasks.getByPath()来定位，参数为任务名，相对路径、绝对路径

```groovy
task hello
println project.hello.name
println tasks['hello'].name
println tasks.getByPath('projectA:hello').path
println tasks.getByPath(':projectA:hello').path
```

### 配置任务
- 使用变量的方式
- 使用闭包的方式

```groovy
Copy myCopy = task(myCopy, type: Copy)
myCopy.from 'xxx'
myCopy.into 'xxx'

task copy(type:Copy) {
	from 'xxx'
	into 'xxx'
}
```

### 替换任务
- 使用overwrite属性

```groovy
task copy(type: Copy)

task copy(overwrite: true) << {
	println 'xxx'
}
```

### 忽略任务

#### 使用谓语
* 使用onlyIf()可以附加一个谓语到task上，该task只有在谓语表达式为true时才能运行

```groovy
 task hello << {
     println 'xxx'
 } 
 hello.onlyIf { !project.hasProperty('skipHello') } 
```

#### 使用StopExecutionException异常
* 如果忽略逻辑无法表达成谓语，则可以使用StopExecutionException。
* 当一个动作抛出这个异常时，该任务的其他动作都将被忽略，构建将继续执行其他的任务

#### 使能/禁止
* 每个任务都用一个enabled标识(默认值是true)
* 当设置为false时，任务的任何动动作都将被阻止

### 忽略任务为最新状态
- 每个任务都有一个inputs, outputs属性，用于定义任务的输入、输出数据
- inputs属性是TaskInputs类型的，outputs属性是TaskOutputs类型。
- 如果任务没有定义outpus，将永远都不会被认为是最新的。
- 对于任务不是文件、比较复杂的场景，TaskOutpus.upToDateWhen()可以允许自定义判断逻辑

```groovy
task transform {
	ext.srcFile = file('xxx')
	ext.destDir = new File(buildDir, 'xxx')
	inputs.file srcFile
	outputs.dir destDir
	doLast {
		println 'xxx'
	}
}
```
#### 原理
- 在任务第一次执行前，gradle会对输入进行快照。任务执行成功后，gradle会对输出进行快照。
- 快照包括文件集合，每个文件内容的hash值。快照数据会被持久化保存
- 在任务以后的执行前，gradle会对输入、输出进行新的快照。
- 若新快照和前一次快照是一样的，gradle会认为输出是最新的，并忽略该任务。

### 任务规则
```groovy
tasks.addRule("Pattern:ping<ID>") { String taskName ->
    if(taskName.startWith("ping")) {
		println 'xxx'
	}
}
```

## 文件操作

### 定位文件
* 使用Project.file()来定位相对于项目目录的文件

```groovy
File configFile = file('src/config.xml')
configFile = file(configFile.absolutePath)
configFile = file(new File('src/config.xml'))
```


### 文件集合
* 表现为FileCollection接口，很多gradle对象都实现了该接口。
* 获取FileCollection实例的一种方法就是使用Project.files()
* 以传递任何个数的对象到这个方法，这些参数将会转换成一组File对象的集合
* files()方法可以接收任何数据类型的对象作为参数，参数将会被求值成相对于项目目录。

```groovy
FileCollection collection = files('src/file1.txt',
							new File('src/file2.txt'),
							['src/file3.txt', 'src/file4.txt'])
```

* 文件集合是可以遍历的，
* 使用as操作符可以转换成多种其他的数据类型
* 使用'+'操作符可以增加其他文件集合对象
* 使用'-'操作符可以减少文件集合对象。

```groovy
collection.each { File file -> println file.name }

Set set = collection.files
Set set2 = collection as Set
List list = collection as List
String path = collection.asPath
File file = collection.singleFile
File file2 = collection as File

def union = collection + files('src/file3.txt')
def different = collection - files('src/file3.txt')
```

* 可以给files()传递一个闭包或Callable实例
* 当集合中的内容被查询时闭包才会被调用
* 闭包的返回值会被转换成一个File对象的集合。

```groovy
File srcDir
collection = files{ srcDir.listFiles() }

srcDir = file('src')
collection.collect { relativePath(it) }.sort().each { println it }

srcDir = file('src2')
collection.collect { relativePath(it) }.sort().each { println it }
```


#### files()传入其他类型

* FileCollection 数据会被扁平化，其内容会被加入到文件集合中
* Task 该任务的输出文件将会被加入到文件集合中
* TaskOutputs 该对象的输出文件将会被加入到文件集合中


### 文件树
* 一个文件树是一个集合，文件按层次方式排列。
* 一个文件树可以表示为一个目录树或一个ZIP文件中的内容
* FileTree为文件树的接口类，该类扩展于FileCollection。
* 使用Project.fileTree()来获取FileTree实例

```groovy
FileTree tree = fileTree(dir: 'src/main')

tree = fileTree('src') {
    include '**/*.java'
}

tree = fileTree(dir: 'src', include: '**/*.java')
tree = fileTree(dir: 'src', includes: ['**/*.java', '**/*.xml'])
tree = fileTree(dir: 'src', include: '**/*.java', exclude: '**/*test*/**')
```

* 可以像使用文件集合那样的使用文件树
```groovy
tree.each {File file ->
    println file
}

FileTree filtered = tree.matching {
    include 'org/gradle/api/**'
}

FileTree sum = tree + fileTree(dir: 'src/test')

tree.visit {element ->
    println "$element.relativePath => $element.file"
}
```

### 按文件树方式使用压缩包的内容
* 通过Project.zipTree(), Project.tarTree()方法来使用压缩包的内容
* 这些方法会返回一个FileTree实例

```groovy
FileTree zip = zipTree('someFile.zip')

FileTree tar = tarTree('someFile.tar')

FileTree someTar = tarTree(resources.gzip('someTar.ext'))
```

### 指定一组输入文件
* gradle中有很多对象都用属性可以接受一组输入文件
* 像JavaCompile任务的source属性。

```groovy
compile {
	source = file('src/main/java')
}

compile {
	source = 'src/main/java'
}

compile {
	source = ['src/main/java', '../shared/java']
}

compile {
	source = fileTree(dir: 'src/main/java')
	              .matching { include 'org/gradle/api/**' }
}

compile {
	source { file('src/test/').listFiles() }
}
```

### 拷贝文件

* 使用Copy任务。可以在拷贝时过滤文件内容，映射文件名。

```groovy
task copyTask(type: Copy) {
    from 'src/main/webapp'
    into 'build/explodedWar'
}
```

* from()方法可以接受任何与files()方法一样的参数
    * 当参数是一个目录时，该目录下的所有文件都会被拷贝到目标目录下
    * 当参数是一个文件时，该文件会被拷贝到目标目录下
    * 当参数时一个不存在的文件时，该参数会被忽略
    * 当参数时一个任务时，该任务的输出文件都会被拷贝，该任务会被自动加入到Copy任务中依赖项中。
* into()方法可以接受任务与file()方法一样的参数。

```groovy
task anotherCopyTask(type: Copy) {
    
    from 'src/main/webapp'
    from 'src/staging/index.html'
    from copyTask
    from copyTaskWithPatterns.outputs
    from zipTree('src/main/assets.zip')
    
    into { getDestDir() }
}

task copyTaskWithPatterns(type: Copy) {
    from 'src/main/webapp'
    into 'build/explodedWar'
    include '**/*.html'
    include '**/*.jsp'
    exclude { details -> details.file.name.endsWith('.html') 
                       && details.file.text.contains('staging') }
}
```

* 也可以使用Project.copy()方法来拷贝文件，
* 该copy方法不是增量进行，其次该copy方法不能使用任务的依赖性
* 如果你使用该copy方法作为任务的动作时，你必须明确的定义任务的inputs和outputs
* **尽可能的使用copy任务是一种更好的选择，因为它支持增量编译和任务依赖**

```groovy
task copyMethodWithExplicitDependencies{    
    inputs.file copyTask
    outputs.dir 'some-dir' 
    doLast {
         copy {            
            from copyTask
            into 'some-dir'
         }
   } 
}
```


#### 重命名文件
```groovy
task rename(type: Copy) {
    from 'src/main/webapp'
    into 'build/explodedWar'
    //使用闭包
    rename { String fileName ->
        fileName.replace('-staging-', '')
    }
    //使用正则表达式
    rename '(.+)-staging-(.+)', '$1$2'
    rename(/(.+)-staging-(.+)/, '$1$2')
}
```

#### 过滤文件
```groovy
import org.apache.tools.ant.filters.FixCrLfFilter
import org.apache.tools.ant.filters.ReplaceTokens
task filter(type: Copy) {
    from 'src/main/webapp'
    into 'build/explodedWar'    
    //替换文件中的属性标记
    expand(copyright: '2009', version: '2.3.1')
    expand(project.properties)
    //使用ant的过滤器
    filter(FixCrLfFilter)
    filter(ReplaceTokens, tokens: [copyright: '2009', version: '2.3.1'])
    //使用闭包来过滤每一行
    filter { String line -> "[$line]" } 
}
```

#### 使用CopySpec类
* copy spec带有层次
* 一个copy spec继承了目标路径、include模式、exclude模式、copy动作、名字映射和过滤

```groovy
task nestedSpecs(type: Copy) {
    into 'build/explodedWar'
    exclude '**/*staging*'
    from('src/dist') {
        include '**/*.html'
    }
    into('libs') {
        from configurations.runtime
     } 
}
```

### 使用sync任务
* sync任务扩展于copy任务
* 任务执行时，将拷贝源文件到目标目录，然后从目标目录中移除所有没有拷贝的文件。

```groovy
task libs(type: Sync) {
    from configurations.runtime
    into "$buildDir/libs"
}
```

### 创建压缩包
```groovy
apply plugin: 'java'
task zip(type: Zip) {
    from 'src/dist'
    into('libs') {
        from configurations.runtime
    } 
}
```

#### 压缩包名字
* 名字格式为projectName-version.type

```groovy
apply plugin: 'java'
version = 1.0
task myZip(type: Zip) {
    from 'somedir'
    baseName = 'customName'
}
println myZip.archiveName   //结果customName-1.0.zip
```

```groovy
apply plugin: 'java'
archivesBaseName = 'gradle'
version = 1.0
task myZip(type: Zip) {
    appendix = 'wrapper'
    classifier = 'src'
    from 'somedir'
}
println myZip.archiveName  //结果gradle-wrapper-1.0-src.zip
```

## 构建的生命周期

* Gradle的核心是基于依赖编程。
* 任务被组织成一个有向无环图。
* 你的构建脚本用于配置这个依赖图，严格意义上他们被叫做构建配置脚本

### 构建阶段

#### 初始化
* Gradle会决定哪些项目成为构建的一部分
* 为每个项目创建出一个Project实例。

#### 配置
* Project对象将会被配置好
* 所有项目的构建脚本也在这个阶段被执行。

#### 执行
* Gradle将会确定一个需要被执行的任务子集，
* Gradle将会执行每一个被选择的任务。

### settings文件
* 除了构建脚本文件，Gradle还定义了一个settings文件
* 文件的默认名为settings.gradle。
* 该文件在初始化阶段被执行
* 一个多项目构建必须要在项目根目录下有一个settings.gradle文件。对于单项目构建该文件是可选的。
* settings文件中的读写属性和调用方法将会被委托给settings对象。

### 多项目构建
* 多项目构建是指在一次gradle执行中会构建超过一个的项目，必须在settings文件中定义项目。

#### 项目定位
* 多项目构建经常表示为有一颗有单一根的树
* 树上的每个元素都代表一个项目，一个项目有一个路径，表示在构建树上的位置
* 在大部分情况下，项目路径表示在文件系统中的物理位置
* 然而该行为是可以配置的
* 项目树在settings.gradle文件中被创建。默认情况下settings文件位于根项目下。

#### 构建树

* settings文件中可以使用一组方法来构建项目树
* 支持层次物理布局、扁平物理布局

##### 层次布局
* include方法参数为项目路径
* 例子：'services:api'会被映射成目录'service/api'

```groovy
	include 'project1', 'project2:build', 'project3:build'
```

##### 扁平布局
* includeFlat方法参数为目录名
* 这些目录是必须要存在，且在根项目的目录下的

```groovy
	includeFlat 'project3', 'project4'
```



### 初始化

#### 查找settings.gradle文件
- 首先在当前目录下查找，该目录被称为master
- 若没有找到，则在父目录下查找
- 若没有找到，则作为单一项目进行构建
- 如果发现settings.gradle，则检查当前项目是否在settings.gradle文件中被定义
    - 若没有则执行一次单项目构建
    - 若找到则执行一次多项目构建

### 构建过程中的响应
* 在整个构建生命周期中都会收到通知作为构建进度
* 这些通知有两种形式：实现一个监听器接口，或者提供一个闭包

#### 项目求值
* 在一个项目求值的前后立刻接收到通知

```groovy
allprojects {
    afterEvaluate { project ->
        if (project.hasTests) {
            println "Adding test task to $project"
            project.task('test') << {
			    println "Running tests for $project"
		    }
		}
	}
}
```

```groovy
gradle.afterProject {project, projectState ->
    if (projectState.failure) {
        println "Evaluation of $project FAILED"
    } else {
        println "Evaluation of $project succeeded"
    }
}
```

#### 任务创建
* 在一个任务被创建并加入项目后立刻接收到一个通知。

```groovy
tasks.whenTaskAdded { task ->
    task.ext.srcDir = 'src/main/java'
}
```

#### 任务执行
* 在一个任务被执行前后立刻接收到一个通知

```groovy
gradle.taskGraph.beforeTask { Task task ->
    println "executing $task ..."
}
gradle.taskGraph.afterTask { Task task, TaskState state ->
    if (state.failure) {
        println "FAILED"
    }
    else {
        println "done"
	} 
}
```


## 初始化脚本

### 概述
* 初始化脚本（简称init scripts）和其他正常的脚本很相似，
* 该脚本是在构建进行前被运行的
* 限制是无法访问buildSrc项目下的类

### 用途
- 设置整个公司范围内的配置，像自定义插件的位置
- 设置当前环境对应的属性，像开发人员的机器和持续集成服务器
- 提供个人信息，像数据库的鉴权信息
- 定义机器香相关的信息，像JDK的安装位置
- 注册构建监听器
- 注册构建日志输出器


### 应用初始化脚本

- 通过命令行指定一个文件，使用参数 -I, --init-script
- 在USER_HOME/.gradle/目录下增加init.gradle文件
- 在USER_HOME/.gradle/init.d目录下增加.gradle后缀的文件
- 在GRADLE_HOME/init.d目录下增加.gradle后缀的文件，在一个gradle发行版本中。
- **如果有多个初始化脚本被发现，他们全部都会被执行，安装前面的顺序。指定目录下的脚本会按照字母顺序运行。**

### 写初始化脚本
* 一个初始化脚本也是一个groovy脚本
* 每个初始化脚本会被分配到一个Gradle实例
* 任务属性引用和方法调用都会被委托给这个Gradle实例。
* 每个初始化脚本也都实现了Script接口。

#### 配置项目

##### 在项目求值前进行额外参数的配置
```groovy
//build.gradle
repositories {
    mavenCentral()
}
task showRepos << {
    println "All repos:"
    println repositories.collect { it.name }
}
```

```groovy
//init.gradle
allprojects {
    repositories {
        mavenLocal()
    }
}
```

输出
**gradle --init-script init.gradle -q showRepos  输出为 [MavenLocal, MavenRepo]**


#### 配置额外依赖
* 初始化脚本也可以定义依赖，可以在initscript()方法中进行。

```groovy
//init.gradle
initscript {
   repositories {
       mavenCentral()
   }
   dependencies {
        classpath group: 'org.apache.commons', name: 'commons-math', version: '2.0'
   } 
}
```

#### 初始化插件
* 插件必须要确保只有一个指定的repo在运行构建时被使用
* 当在初始化脚本中应用一个插件时，Gradle会实例化这个插件，调用实例的Plugin.apply()方法。

```groovy
//init.gradle
apply plugin:EnterpriseRepositoryPlugin
class EnterpriseRepositoryPlugin implements Plugin<Gradle> {
     
     void apply(Gradle gradle) {
     .....    
     }
}
```



## 日志

### 日志水平
| 水平     |   用于 |
| :--------  | :------------  |
| ERROR      | 错误信息        |
| QUIET      | 非常重要的信息  |
| WARNING    | 警告信息   |
| LIFECYCLE  | 运行过程信息  |
| INFO  | 一般信息 |
| DEBUG  | 调试信息 |

### 日志水平的选择
|选项     |  输出日志水平 |
|:-------  | :----------- |
|无日志参数  | LIFECYCLE及更高|
|-q或--quiet | QUIET及更高|
|-i或--info  | INFO及更高|
|-d或--debug | DEBUG及更高|

### 写日志信息

* gradle会重定向任何标准输出的信息到日志系统中的QUIET水平

```groovy
println 'A message which is logged at QUIET level'
```

* 提供了一个logger属性
* 是一个Logger实例。该接口扩展于SLF4J，并增加了一个方法。

```groovy
logger.quiet('An info log message which is always logged.')
logger.error('An error log message.')
logger.warn('A warning log message.')
logger.lifecycle('A lifecycle info log message.')
logger.info('An info log message.')
logger.debug('A debug log message.')
logger.trace('A trace log message.')
```

* 可以修改gradle的日志系统

```groovy
import org.slf4j.Logger
import org.slf4j.LoggerFactory
Logger slf4jLogger = LoggerFactory.getLogger('some-logger')
slf4jLogger.info('An info log message logged using SLF4j')
```


### 外部工具和库的日志

* 默认情况下，gradle会重定向标准输出到QUIET水平，标准错误到ERROR水平
* 项目对象提供了一个LoggingManager来进行配置。

```groovy
logging.captureStandardOutput LogLevel.INFO
println 'A message which is logged at INFO level'
```

* 任务也提供了一个LoggingManager对象

```groovy
task logInfo {
    logging.captureStandardOutput LogLevel.INFO
    doFirst {
        println 'A task message which is logged at INFO level'
    }
}
```

### 改变Gradle日志

* 需要使用Gradle.useLogger()方法

```groovy
useLogger(new CustomEventLogger())
class CustomEventLogger extends BuildAdapter implements TaskExecutionListener {
    public void beforeExecute(Task task) {
        println "[$task.name]"
    }
    public void afterExecute(Task task, TaskState state) {
        println()
    }
    public void buildFinished(BuildResult result) {
        println 'build completed'
        if (result.failure != null) {
		    result.failure.printStackTrace()
        } 
     }
}
```

* 可以实现以下监听器接口中的任何一个。
    - BuildListener
    - ProjectEvaluationListener
    - TaskExecutionGraphListener
    - TaskExecutionListener
    - TaskActionListener


## 构建环境

### 使用gradle.properties

#### 载入顺序
- 载入项目目录下的gradle.properties文件
- 载入gradle用户目录下的gradle.properties文件
- 使用系统属性

#### 配置属性
- org.gradle.daemon 设置true会使用gradle守护进程来运行构建。对于本地开发者这是一个非常受欢迎的特性。
- org.gradle.java.home 为gradle构建进程设置java安装目录。该值可以设置成jdk, jre的安装目录，然而设置jdk更安全一些。
- org.gradle.jvmargs 给守护进程设置jvmargs参数。该设置对于调整内存设置很有用。
- org.gradle.configureondemand 使gradle在配置项目时可以进行选择。只有相关的项目才会被配置，用于在比较大的多项目中进行快速构建。
- org.gradle.parallel 使gradle运行在并行构建模式。





## 插件

* Gradle核心有意的只提供了一点功能
* 其他所有有用的特性，像编译java代码，都使用由**插件**提供的

### 插件类型

* 脚本插件: 额外的构建脚本，用于更进一步的配置构建过程
* 二进制插件: 是一些实现了Plugin接口的类，使用一种编程式方法来管理构建


### 应用插件
* 插件所说的被应用，是指调用了Project.apply()方法

#### 脚本插件
* 可以通过一个本地文件系统的、远程位置的脚本文件来应用
* 文件系统位置必须是相对于项目目录的，远程位置必须是一个HTTP的URL

```groovy
apply from: 'other.gradle'
```

#### 二进制插件

```groovy
apply plugin: 'java'  //使用短名

apply plugin: JavaPlugin  //使用类型

```

##### 二进制插件的位置
* 定义插件，使用构建脚本中的内部类
* 定义插件，使用buildSrc目录下的源文件
* 从外部jar中引入插件，作为脚本依赖
* 使用插件DSL，引入插件


### 使用插件DSL应用插件

```groovy
plugins {
	id 'java' //应用一个核心插件
}

plugins {
	id 'com.jfrog.bintray' version '0.4.1' //应用一个社区插件
}
```

### 插件做了什么
* 向项目添加任务
* 使用默认值预先配置任务
* 向项目添加依赖配置
* 使用扩展向已存在的类添加属性和方法

```groovy
apply plugin: 'java'
task show << {
    println relativePath(compileJava.destinationDir)
    println relativePath(processResources.destinationDir)
}
```


## 依赖管理

### 定义依赖

```groovy
apply plugin: 'java'
repositories {
    mavenCentral()
}
dependencies {
    compile group: 'org.hibernate', name: 'hibernate-core', version: '3.6.7.Final'
    testCompile group: 'junit', name: 'junit', version: '4.+'
}
```

### 依赖配置

* java插件定义了一些标准配置，这些配置表明了插件所使用的类路径。
* **compile** 这些依赖会在编译项目产品源代码时被要求。
* **runtime** 这些依赖会在运行时，由产品类所依赖。默认也会包含编译时的依赖。
* **testCompile** 这些依赖会在编译项目测试代码是被请求。默认会包括已编译的产品类和编译时依赖。
* **testRuntime** 这些依赖会在运行测试是被请求。默认会包括编译、运行、测试编译依赖。


### 外部依赖

* 一个外部依赖使用 组、名字、版本属性来标识
* 缩写形式为‘组:名字:版本’

```groovy
dependencies {
     compile group: 'org.hibernate', name: 'hibernate-core', version: '3.6.7.Final'
}
```

```groovy
dependencies {
    compile 'org.hibernate:hibernate-core:3.6.7.Final'
}
```

### 仓库

```groovy 
repositories {
    mavenCentral() //定义maven中心库
}

repositories {
    maven {
        url "http://repo.mycompany.com/maven2" //远程maven库
    }
}

repositories {
    ivy {
        // URL can refer to a local directory  
        url "../local-repo"  //文件系统
    }
}

```





