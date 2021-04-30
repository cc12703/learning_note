


# 如何使用7个简单步骤加强Linux服务器的安全

* https://medium.com/servers-101/how-to-secure-your-linux-server-6026cfcdefd8

[TOC]


## 概述
* 这并不是一个完整的安全引导。但是可以帮助你预防90%的著名的后端攻击者：像暴力破解、DDos

### 准备工作
1. 需要有一台Linux服务器
1. 需要对命令行有一个基础的理解


## 步骤

### 配置SSH键
* 你必须使用密码或者SSH键登录，来访问远端服务器
* 使用密码的问题就是密码太容易被暴力破解。而且在需要访问服务器是必须要键入密码
* 为了避免这些缺点，你可以启用SSH鉴权。由于黑客无法暴力破解，这个比密码更安全
    * 由于不在需要输入密码，连接服务器时会更简单和快速

#### 操作流程
1. 在本机上生成SSH密码对
    ```bash
   ssh-keygen
    ```
    说明
        * 密码会被存储在文件中
1. 向服务器增加公钥
    ```bash
    ssh-copy-id username@remote_host
    ```
1. 使用以下命令进行登录
    ```bash
    ssh usrname@remote_host
    ```

### 保持系统时间是最新的
* 大多数安全协议会利用你的系统时间来运行周期性任务，记录日志，执行其他关键性任务
* 如果你的系统时间不正确，会对你的服务器有负面的影响
* 为了避免这些情况发生，你可以安装NTP客户端
    * 该客户端可以通过与全局的NTP服务器进行同步来保持你系统时间是最新的

#### 操作流程
1. 安装
    ```bash
    sudo apt install ntp
    ```


### 查看激活的端口号
* 服务器上的应用会导出特定的端口，以使网络中的其他应用可以访问该应用
* 黑客也会在服务器上安装后门程序并导出端口号，来控制服务器
* 为此，你不希望服务器在监听一些你不知道的端口号

#### 操作步骤
1. 查看端口号
    ```bash
    sudo ss -lntup
    ```
说明
* 试着去发现和追踪潜在的不安全的服务和进程
* 可以通过查看[坏TCP/IP端口号](https://www.garykessler.net/library/bad_ports.html)来开始


### 设置防火墙
* 防火墙允许你阻止你服务器中特定端口的输入和输出
* 为此我经常使用UFW（简单的防火墙）

#### UFW配置规则
* 允许或拒绝
* 输入流量或输出流量
* 流向目标或从目标流入
* 特定或全部端口号

#### 操作步骤
1. 安装UFW
    ```bash
    sudo apt-get install ufw
    ```
1. 拒绝所有输出流量
    ```bash
    sudo ufw default deny outgoing comment 'deny all outgoing traffic'
    ```
1. 允许所有的输出流量
    ```bash
    sudo ufw default allow outgoing comment 'allow all outgoing traffic'
    ```
1. 拒绝所有的输入流量
    ```bash
    sudo ufw default deny incoming comment 'deny all incoming traffic'
    ```
1. 除外SSH连接，这样你就可以访问系统了
    ```bash
    sudo ufw limit in ssh comment 'allow SSH connections in'
    ```
1. 如果你将UFW配置成拒绝所有输出流量，不要忘记允许你需要的特定流量
    ```bash
    # 允许端口号53的流量 -- DNS
    sudo ufw allow out 53 comment 'allow DNS calls out'
    # 允许端口号123的流量
    sudo ufw allow out 123 comment 'allow NTP out'
    # 允许HTTP, HTTPS, FTP上的输出流量
    # apt需要这些来连接源服务器
    sudo ufw allow out http comment 'allow HTTP traffic out'
    sudo ufw allow out https comment 'allow HTTPS traffic out'
    sudo ufw allow out ftp comment 'allow FTP traffic out'
    # 允许whois服务
    sudo ufw allow out whois comment 'allow whois'
    # 允许端口68的流量 -- DHCP 客户端
    # 如果你使用了DHCP，这必须需要该端口号
    sudo ufw allow out 68 comment 'allow the DHCP client to update'
    ```
1. 允许端口号99的流量
    ```bash
    sudo ufw deny 99
    ```
1. 最后启动UFW
    ```bash
    sudo ufw enable
    ```
1. 查看UFW状态
    ```bash
    sudo ufw status
    ```


### 阻止自动攻击
* 以下两个工具可以阻止大部分的自动攻击
    * PSAD
    * Fail2Ban

#### 两者的不同点
* 端口号提供了对服务器上应用的访问
* 攻击者可以扫描你服务上公开的端口
* PSA通过监控网络活动来侦测和可选的阻止像扫描和其他类型的可疑流量，像DDoS或OS指纹识别尝试
* Fail2Ban通过扫描不同应用的日志文件，自动禁止那些有恶意行为的IP（像自动登录尝试）



### 安装logwatch
* 服务器中的应用经常会在日志文件中保存日志
* 除非你想手动监控这些日志，不然你就需要安装logwatch
* logwatch会扫描系统的日志文件并对其进行分析和总结
* 你可以在命令行直接运行它或者在一个循环调度器着中定时运行它
* 你可以配置logwatch，给你发送一封日志文件的日结报告

#### 配置
* logwatch使用service文件来获取如何读取和总结一个日志文件
    * service文件位置：/usr/share/logwatch/scripts/services
* 配置文件 /usr/share/logwatch/default.conf/logwatch.conf


#### 操作流程
1. 安装logwatch
    ```bash
    apt-get install logwatch
    ```
1. 直接运行
    ```bash
    sudo /usr/sbin/logwatch --output stdout --format text --range yesterday --service all
    ```
1. 配置logwatch,发送每日结报告邮件
    ```bash
    /usr/sbin/logwatch --output mail --format html --mailto root --range yesterday --service all
    ```


### 执行安全审计
* 在加固你的服务器后，你需要执行安全审计来发现遗漏的安全漏洞
* 可以使用Lynis，一个开源软件可以执行以下功能
    * 安全审计
    * 合规性测试（PCI,HIPAA,SOx）
    * 渗透测试
    * 脆弱性侦测
    * 系统锻炼
* [参考网页](https://cisofy.com/lynis/)

#### 操作步骤
1. 下载源码
    ```bash
    git clone https://github.com/CISOfy/lynis
    ```
1. 切换到代码目录
    ```bash
    cd lynis
    ```
1. 运行审计
    ```bash
    lynis audit system
    ```

