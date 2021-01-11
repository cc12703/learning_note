
# Docker使用

[TOC]


## 命令行

### 概述
* 配置优先级：命令行选项 高于 环境变量  高于  配置文件

### 选项
* --config  指定配置文件目录
* --debug (-D) 启用调试模式
* --host (-H)  服务器的连接方式 【tcp://host:port or unix:///path/to/socket or  fd://socketfd】
* --log-level(-l) 指定输出日志的级别 【debug or info or warn or error】

### 环境变量
* DOCKER_API_VERSION  要用的API的版本号
* DOCKER_CONFIG  指定配置文件目录
* DOCKER_DRIVER 要使用的graph驱动
* DOCKER_HOST   服务器的连接方式 
* DOCKER_TMPDIR  临时文件目录

### 配置文件
* 默认情况下，配置文件保存在$HOME目录下的.docker目录中。
* docker.json 可以修改，数据格式为json。

### container子命令

#### 概述
* **docker container \<command\>**
* 功能：用于管理容器

#### attach命令
* **docker container attach  \<option\> \<container id or container name\>**
* 连接本地的标准输入、标准输出、错误输出到一个已运行的容器上

##### 选项
* --detach-keys  重写从容器断开的按键序列
* --no-stdin   不连接上标准输入

##### 说明
* ctrl-c 用于停止容器，该操作会发送SIGKILL给容器
* ctrl-p ctrl-q 用于从已连接的容器上断开


#### commit命令
* **docker container commit \<option\> \<container id\> \<resp:tag\>**
* 从一个容器中创建一个新镜像

##### 选项
* --author -a   指定作者
* --change -c  指定Dockerfile指令用于创建镜像
* --message -m  指定提交的信息
* --pause -p  是否在提交过程中暂停容器

##### 说明
* 提交操作不会包含数据卷中的任何数据
* 默认情况下，提交操作时容器中的任何进程都会被暂停

#### cp命令
* **docker container cp \<option\> container:src-path  dest-path**
* **docker container cp \<option\> src-path   container:dest-path**
* 在本地文件系统和容器之间拷贝文件和目录

##### 选项
* --archive  -a  归档模式（拷贝所有的uid, gid信息）


#### create命令
* **docker container create \<option\>  image \<command\> \<arg\>**
* 创建一个新容器

##### 选项
* --cidfile  写入容器ID到文件
* --device  增加一个主机设备到容器

##### 说明
* 该命令会在指定镜像上创建一个可写的容器层。会在标准输出上打印出容器ID


#### diff命令
* **docker container diff  container-id or container-name**
* 在容器文件系统中检查文件，目录的变动

##### 说明
跟踪三种状态：
* A  新增加
* D  被删除
* C  被修改

#### exec命令
* **docker container exec \<option\> container-id  command \<arg\>**
* 在已运行的容器中运行命令

##### 选项
* --detach -d  脱机模式，在后台运行命令
* --interactive  -i  保持标准输入打开，就算没有attach上
* --privileged 给命令附加上额外的特权
* --tty -t  分配一个虚拟终端
* --user  -u  指定用户名或者UID
* --workdir  -w  容器中的工作目录

##### 说明
* 命令在容器主进程（PID 1）上运行
* 容器重启后，该命令不会重新运行


#### export命令
* **docker container export \<option\> container-id**
* 将容器文件系统导出为一个归档文件

##### 选项
* --output -o   写入到文件中

##### 说明
* 该命令不会导出容器中的数据卷


#### start命令
* **docker container start \<option\> container-id  \<container-id\> ...**
* 启动一个，多个已停止的容器

##### 选项
* --attach  -a   连接上标准输出、错误输出。转发系统信号
* --interactive  -i  连接上标准输入


### 镜像子命令

* **docker image \<command\>**
* 用于管理镜像

#### build命令
* **docker image build \<option\> \<path or url\>**
* 从dockerfile文件中构建出镜像

##### 选项
* --build-arg 设置构建时变量
* --label   设置镜像的元数据
* --add-host 添加自定义的主机名到IP地址的映射（格式：name:ip）
* --cache-from  指定镜像的缓存源
* --no-cache 构建时不使用缓存
* --compress  使用gzip压缩构建上下文
* --file (-f)  指定dockerfile文件
* --force-rm 强制移除中间步骤容器
* --tag  设置镜像的tag名字  （格式：name:tag）
* --progress  设置进度输出格式 （auto, plain, tty）

##### 说明
* 构建命令从一个Dockerfile 和 一个 环境上下文 中构建出镜像
* 构建上下文是指一组由 path 和 url 指定的文件，构建过程可以引用该上下文中的任何文件

##### 例子
```
docker build https://github.com/docker/rootfs.git#container:docker   从git仓库中构建
docker build .   从当前目录中构建
docker build -f ctx/Dockerfile http://server/context.tar.gz  从压缩包中构建
docker build - < Dockerfile   从文本文件中构建
```

### 有用的命令

#### 批量删除无用镜像
**docker rmi $( docker images -f dangling=true)**

#### 登录已运行容器
**docker exec -it container_name /bin/bash**


## Dockerfile

### 总体
* 使用’#’作为注释
* 分成四个部分：基础镜像信息、维护者信息、镜像操作指令、容器启动指令

### 指令

#### FROM
* 格式：FROM \<image\>:\<tag\>
* 作用：Dockerfile的第一条指令，用于指定要继承的镜像

#### MAINTAINER
* 格式: MAINTAINER \<name\>
* 作用：指定维护者信息

#### RUN
* 格式1：RUN \<command\>  
* 格式2：RUN ['cmd', 'param1', 'param2']
* 作用：执行shell命令，翻译成 /bin/sh -c "xxx"

#### EXPOSE
* 格式：EXPOSE \<port\> \<port\>/\<protocol\>  ...
* 作用：暴露端口号

#### CMD
* 格式1： CMD [ 'executable', 'param']    使用exec执行，推荐使用
* 格式2： CMD command param    在/bin/sh中执行，用于需要交互的应用
* 格式3： CMD ['param'] 提供给ENTRYPOINT的默认参数
* 作用：指定启动容器时执行的命令，每个Dockerfile只能有一条CMD指令，可以在启动容器时覆盖这条命令

#### ENTRYPOINT
* 格式1： ENTRYPOINT ['executable' ,  'param']
* 格式2： ENTRYPOINT command param
* 作用：用于设置容器启动时要执行的命令及其参数。该命令一定会被执行

#### VOLUME
* 格式：VOLUME ['/data']
* 作用：创建一个挂载点

#### WORKDIR
格式：WORKDIR /path
作用：设置当前的工作目录

#### ENV
* 格式：ENV \<key\>  \<value\>
* 作用：指定一个环境变量，会被后续RUN指令使用，且在容器运行时有效

#### ADD
*格式：ADD \<src\> \<dst\>
* 作用：复制指定的src到容器中的dst, src可以是目录，URL，tar文件

#### COPY
* 格式：COPY \<src\> \<dst\>
* 作用：复制本地主机的src到容器中的dst

#### LABEL
* 格式：LABEL \<key\>=\<value\> \<key\>=\<value\> ...
* 作用：在镜像中增加元信息

### 生成镜像过程
1. 解析Dockerfile， 找到基础镜像
2. 以基础镜像为基础创建一个容器
3. 在容器中顺序执行Dockerfile中的命令
4. 记录非RUN命令，以便在启动时执行
5. 属性命令记录在镜像的属性中
6. 指令执行完后，commit该容器为新的镜像

