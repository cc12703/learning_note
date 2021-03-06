[TOC]

# SSL-TLS协议

## 总体

* SSL (Secure Socket Layout)，最初有网景公司开发
* TLS (Transport Layout Security)，由IETF在SSL3.0基础上设计的协议
* 所有信息都是加密的，第三方无法窃听
* 具有校验机制，被篡改后双方会立刻发现
* 配备身份证书，防止身份被冒充


## TLS

* 由记录协议（record protocol）和握手协议（handshake protocol）叠加而成
* 记录协议位于底层，负责进行加密

### 主流程
1. 客户端向服务器索要并验证公钥
2. 双方协商生成’对话密钥‘
3. 双方采用’对话密钥‘进行加密通信

### 记录协议

* 负责使用对称密码对消息进行加密处理

#### 流程
1. 分割消息成多个较短的片段（fragment）
2. 对片段进行压缩
3. 对片段加上消息认证码，附加上片段的编号
4. 使用对称密码对片段进行加密
5. 给片段加上报文头


### 握手协议

* 负责生成共享密钥、交换证书

#### 流程

##### 客户端发出请求
* 发送client hello消息

###### 提供信息
* 支持的协议版本
* 客户端生成的随机数
* 支持的加密方法、压缩方法


##### 服务器回应
* 发送server hello消息
* 发送certificate消息。包含服务器证书清单
* 发送server key exchange消息。
* 发送certificate request消息。请求客户端的证书
* 发送server hello done消息

###### 提供信息
* 确认使用的协议版本
* 服务器生成的随机数
* 确认使用的加密方法、压缩方法
* 服务器证书


##### 客户端回应
* 发送certificate消息。包含客户端证书清单
* 发送client key exchange消息，包含预备主密码
* 发送certificate verify消息，证明是客户端证书的持有者
* 发送change cipher spec消息，切换密码
* 发送finished消息，加密后发送


##### 服务器最后回应
* 发送change cipher spec消息，切换密码
* 发送finished消息，加密后发送



#### 作用
1. 客户端获得了服务器的合法公钥，完成了服务器认证
2. 服务器获得了客户端的合法公钥，完成了客户端认证
3. 客户端、服务器生成了通信中使用的共享密钥
4. 客户端、服务器生成了消息认证码中使用的共享密钥


#### 注意点
* 生成对话密钥一共需要三个随机数
* 服务器公钥、私钥只用于加密、解密’对话密钥‘
* 服务器公钥放在服务器的数字证书中


### 主密码

* 客户端和服务器协商出来的一个48字节的数值

#### 如何计算
1. 预备主密码：客户端发送给服务器
2. 客户端随机数
3. 服务器随机数

#### 目的
1. 生成对称密码的密钥
2. 生成消息认证码的密钥
3. 生成对称密码的CBC模式所使用的初始化向量



### 代理

#### 流程
1. client会与proxy建立tcp连接，并发送connect消息
2. proxy会与server建立tcp连接
3. client与server会建立一个端到端的安全通道
4. proxy变成一个内容转发器，内容无法解密和改动