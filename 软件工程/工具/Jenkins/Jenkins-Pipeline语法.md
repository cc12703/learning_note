


# Jenkins-Pipeline语法

## 概述

* 所有有效的声明式语句都必须包含在一个pipeline的块中
* 基本语句和表达式都遵循Groovy语法

### 例子
```groovy
pipeline {
    /* insert Declarative Pipeline here */
}
```

### 例外
* pipeline必须是一个块(block)
* 语句不使用分号进行分隔，每个语句是一行
* 块必须由 段(sections)，指令(directives)，步骤(steps)或者 赋值语句构成
* 所有的属性引用都会被当成一次无参数的函数调用


## 段(sections)
* 段在pipeline中包含一个或者多个指令或者步骤

### 代理(agent)
* 用来指定整个pipeline或者一个运行期将会在jenkins的那个环境中运行
* 必须配置的
* 可用于顶层pipeline块 和 每个stage块

#### 参数值
##### any
* 在有效的代理上运行

##### none
* 使用在顶层的pipeline时，整个pipeline运行时将不会分配到一个全局的代理。
* 每个stage段需要指定自己的agent

##### label
* 在指定名字的代理上运行

```groovy
agent {
	label 'my-label-name'
}
```

##### node
* 类似于label，允许使用更多的配置项

```groovy
agent {
	node {
		label 'my-label-name'
		customWorkspace 'path'
	}
}
```

##### docker
* 在docker容器中运行
* 可以指定镜像，dockerfile文件
* 可以指定镜像源

```groovy
agent {
    docker {
        image 'maven:3-alpine'
        label 'my-defined-label'
        args  '-v /tmp:/tmp'  //该参数会传递给docker run
    }
}

agent {
    docker {
        image 'myregistry.com/node'
        label 'my-defined-label'
        registryUrl 'https://myregistry.com/'
        registryCredentialsId 'myPredefinedCredentialsInJenkins'
    }
}

agent {
    //等于命令行 docker build -f Dockerfile.build --build-arg version=1.0.2 ./build/
    dockerfile {
        filename 'Dockerfile.build'
        dir 'build'
        label 'my-defined-label'
        additionalBuildArgs  '--build-arg version=1.0.2'
        args '-v /tmp:/tmp'
    }
}
```

### 执行后操作(post)

* 用于定义多个步骤，在pipeline或者stage执行完成后运行
* 支持任何post条件块
* 可选配置的
* 可用于顶层pipeline块 和 每个stage块

#### 条件值
* always 不管运行完成状态是什么，都会执行
* changed 只有当前的完成状态和前一次不一样，才会执行
* fixed 只有当前的完成状态是 ‘成功’，前一次是 ‘不稳定’ 时，才会执行
* regression 只有当前的完成状态是 ‘失败’，‘不稳定’，‘被终止’，前一次是 ‘成功’ 时，才会执行
* aborted 只有当前的完成状态是 ‘被终止'，才会执行。UI界面上会被标注成灰色
* failure 只有当前的完成状态是 ‘失败’，才会执行。UI界面上会被标注成红色
* success 只有当前的完成状态是 ‘成功’，才会执行。UI界面上会被标注成绿色
* unstable 只有当前的完成状态是 ‘不稳定’，才会执行。UI界面上会被标注成黄色
* cleanup 在其他每一个条件都被执行后以后，才会执行。

#### 例子
```groovy
   post { 
        always { 
            echo 'I will always say Hello again!'
        }
    }
```

### 阶段(stages)
* 包含了一个或多个stage指令
* 用于放置一组由pipeline描述的工作。
* 至少要包含一个stage指令
* 每个stage包含持续交付过程中的一部分（构建、测试、部署）

#### 例子
```groovy
    stages { 
        stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
    }
```

### 步骤(steps)
* 定义了一连串可以执行的步骤

#### 例子
```groovy
      stage('Example') {
            steps {
                echo 'Hello World'
            }
        }
```

## 指令(Directives)

### 环境(environment)
* 可以指定一组键值对，来定义成为所有步骤执行时的环境变量。
* 该指令支持credentials() 方法，用于读取在Jenkins中预定义的凭证信息

#### 例子
```groovy
pipeline {
    agent any
    environment { 
        CC = 'clang'
    }
    stages {
        stage('Example') {
            environment { 
                AN_ACCESS_KEY = credentials('my-prefined-secret-text') 
            }
            steps {
                sh 'printenv'
            }
        }
    }
}
```

### 配置项(options)
* 用于配置一些pipeline相关的选项
* 这些选项由pipeline提供，或者相关的插件提供。

#### 有效的配置项

##### buildDiscarder

##### checkoutToSubdirectory
* 从版本控制系统中签出代码时，放入到工作区的子目录中

```groovy
options { 
	checkoutToSubdirectory('foo') 
}
```

##### timeout
* 用于设置pipeline运行的超时时间，超时后jenkins将中止pipeline的运行

```groovy
options { 
	timeout(time: 1, unit: 'HOURS')
}
```


### 参数(parameters)
* 用于配置一些由用户提供的参数
* 这些参数可以在pipeline运行时使用 params 对象来引用

#### 有效参数

##### string
```groovy
parameters { string(name: 'DEPLOY_ENV', defaultValue: 'staging', description: '') }
```

##### text
* 可以配置多行文本

```groovy
parameters { text(name: 'DEPLOY_TEXT', defaultValue: 'One\nTwo\nThree\n', description: '') }
```

##### bool
```groovy
 parameters { booleanParam(name: 'DEBUG_BUILD', defaultValue: true, description: '') }
```

##### choice
```groovy
parameters { choice(name: 'CHOICES', choices: ['one', 'two', 'three'], description: '') }
```

##### file
* 用于上传一些文件

```groovy
parameters { file(name: 'FILE', description: 'Some file to upload') }
```

##### password
* 用于输入密码

```groovy
parameters { password(name: 'PASSWORD', defaultValue: 'SECRET', description: 'A secret password') }
```


### 触发(triggers)
* 用于定义一些自动方式来重复触发pipeline

#### 配置值
##### cron
* 定时触发，使用cron语法

```groovy
triggers { cron('H */4 * * 1-5') }
```

##### poll-scm
* 定时检查代码是否有变动，如果有变动则触发

```groovy
triggers { pollSCM('H */4 * * 1-5') }
```

##### upstream
* 定义一组job名字，当任何job的执行结果小于指定的阀值时，则触发

```groovy
triggers { upstream(upstreamProjects: 'job1,job2', threshold: hudson.model.Result.SUCCESS) }
```

#### cron语法
* 每行包含五个字段，使用TAB或者空格分离

##### 字段
* MINUTE  每小时的指定分钟( 0 - 59)
* HOUR  每天的指定小时 (0 - 23)
* DOM   每月的指定天 （1 - 31）
* MONTH  指定月 （1 - 12）
* DOW  每星期的指定天 （ 0 - 7）

##### 值
* \*  任何值
* M-N   一个范围的值
* M-N/X 或 \*/X   按X指定的间隔进行
* A,B,...,Z   多个值

##### 符号
* H  

#### 例子
```groovy
triggers{ cron('H/15 * * * *') }  每15分钟运行一次

triggers{ cron('H(0-29)/10 * * * *') }  每10分钟运行一次，在0-29分钟内

triggers{ cron('H H(9-16)/2 * * 1-5') } 每2小时运行一次，工作日的上午9点到下午5点
```

### 阶段(stage)
* 在stages段中使用
* 用于包含 一个steps段，一个可选的agent段，一些stage相关的指令

### 工具(tools)
* 定义一些工具，会自动安装并添加到PATH中

#### 例子
```groovy
pipeline {
    agent any
    tools {
        maven 'apache-maven-3.0.1'    //名字必须要在jenkins中定义过
    }
    stages {
        stage('Example') {
            steps {
                sh 'mvn --version'
            }
        }
    }
}
```

### 条件(when)
* 用于让stage按指定的条件来执行
* 至少要包含一个条件，如果有多个条件，则默认是所有条件都满足才会执行

#### 内建条件

##### branch
* 当构建的分支匹配时触发

```groovy
when { branch 'master' }
```

##### buildingTag
* 当构建生成一个tag时触发


## 参考资料

* https://jenkins.io/doc/book/pipeline/syntax/