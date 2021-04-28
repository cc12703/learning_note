

# Github_Actions


[TOC]


## 概述

### 是什么
* actions指持续集成中的各种操作
    * 抓取代码、运行测试、登录远程服务器、发布到第三方服务
* 每个action就是一个独立脚本，可以存放到代码仓库中


### 如何发现
* 官方市场 https://github.com/marketplace?type=actions
* awesome-actions  https://github.com/sdras/awesome-actions


### 如何引用
* 格式： userName/repoName@verName
* 例子
    * actions/setup-node@74bc508
    * actions/setup-node@v1.0
    * actions/setup-node@master


## 概念

### workflow
* 持续集成的一次运行过程

### job
* 持续集成运行中，完成的任务
* 每个workflow由多个job构成

### step
* 每个job由多个step构成

### action
* 每个step可以依次执行多个action


## 配置文件

### 概述
* 位置：仓库的 .github/workflows目录
* 格式： YAML格式

### 参考文档
* 语法文档：https://help.github.com/en/articles/workflow-syntax-for-github-actions
* 事件文档：https://help.github.com/en/articles/events-that-trigger-workflows

### 基本字段
* name workflow的名字
* on 触发workflowy运行的条件
    * 示例
        ```yaml
        on: push
        on: {push, pull_request}
        on:  #当master分支发生pull事件时触发
            push:
                branches:
                    - master
        ```
* jobs.name 定义任务名字
    * 格式： jobs.<job_id>.name
    * 示例
        ```yaml
        jobs:
            my_first_job:
                name: My first job
            my_second_job:
                name: My second job
        ```
* jobs.needs 定义任务依赖关系
    * 格式：jobs.<job_id>.needs
    * 示例
        ```yaml
        jobs:
            job1:
            job2:
                needs: job1
            job3:
                needs: [job1, job2]
        ```
* jobs.runs-on 定义任务运行的虚拟机环境
    * 格式：jobs.<job_id>.runs-on
    * 示例
        ```yaml
        jobs:
            job1:
                runs-on: ubuntu-18.04
        ```
* jobs.steps 定义任务运行的步骤
    * 格式：Jobs.<job_id>.steps
    * 字段：
        * name 名字
        * run 运行的命令和action
        * env 需要的环境变量



### 示例
```yaml
name: Greeting from Mona
on: push

jobs:
  my-job:
    name: My Job
    runs-on: ubuntu-latest
    steps:
    - name: Print a greeting
      env:
        MY_VAR: Hi there! My name is
        FIRST_NAME: Mona
        MIDDLE_NAME: The
        LAST_NAME: Octocat
      run: |
        echo $MY_VAR $FIRST_NAME $MIDDLE_NAME $LAST_NAME.
```