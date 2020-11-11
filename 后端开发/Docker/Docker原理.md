[TOC]

# Docker原理


## 容器原理

### 概述

* **容器 = cgroup + namespace + rootfs + 容器引擎**
* cgroup  资源控制
* namespace  访问隔离
* rootfs 文件系统隔离
* 容器引擎   生命周期控制

### cgroup

#### 概述
* control group, 是linux内裤的一个特性，用于限制和隔离一组进程对系统资源的使用。
* 资源包括：CPU， 内存，IO，网络带宽
* 通过cgroupfs虚拟文件系统来提供接口

#### 子系统
* **cpuset**: 为一组进程分配指定的CPU和内存节点
* **cpu**: 用于限制进行的CPU占用率，包括：CPU比重分配、CPU带宽限制、实时进程的CPU带宽限制
* **cpuacct**: 用于统计各个cgroup的CPU使用情况
* **memory**: 用于限制cgroup所能使用的内存上限
* **blkio**: 用于限制cgroup的block IO带宽
* **devices**: 控制cgroup的进程对哪些设备有访问权限


### namespace

#### 概述
* 将内核的全局资源做封装, 使得每个namespace都有一份独立的资源
* 对同一种资源的使用不会相互干扰
* 通过clone, setns, unshare来实现

#### 子系统
* **UTS**: 用于隔离主机名和域名
* **IPC**: 用于隔离进程间通信
* **PID**: 用于隔离进程PID号
* **Mount**: 用于隔离文件系统的挂载点
* **Network**: 用于隔离网络相关的资源, 包括：网络设备、IP地址、路由表、端口号、/proc/net目录
* **User**: 用于隔离用户ID和组ID


## 镜像原理

### 概述

* 把应用的运行时环境和应用打包在一起，解决了部署环境依赖问题
* 引入分层文件系统，解决了空间利用问题


### 标识

#### 镜像名
* **格式：remote-image-hub/namespace/repository:tag**
* remote-image-hub 存储镜像的web服务器地址，默认为Docker官方镜像库
* namespace  表示一个用户、组织中所有镜像的集合
* repository  仓库名，一个仓库可以包括多个镜像，通过tag来区分
* tag 用于区分同一类镜像的不同版本

#### 识别号
* layer 镜像有一系列层组成，每层都用64bit的十六进制数表示
* image id  镜像最上层的layer id


### 数据组织

#### 概述
* 包括：数据 和 元数据
* 数据：由一层层的image layer组成
* 元数据：一些json文件，描述image layer之间的关系及容器的配置信息

#### overlay存储驱动实例
* **目录 /var/lib/docker**
* ./graph  存储image layer的元数据
* ./overlay   存储image layer数据
* ./repositories-overlay  存储总体信息
* ./trust     存储验证相关信息
* ./volumes    存储数据劵相关信息

#### 还原image
1. 通过repositories-overlay获取image对应的 layer id
2. 根据layer对应的元数据获取出image所包含的所有层，及层与层之间的关系
3. 使用联合挂载技术还原出容器所需要的rootfs和基本配置信息


### 联合挂载

* 将多个目录挂载到同一个目录上，对外呈现这些目录的联合

#### 写时复制
* **覆盖**: 上层的文件会覆盖同名的下层文件
* **删除**: 不直接删除文件，只是在上层建立一个同名的主次设备号都为0的字符设备