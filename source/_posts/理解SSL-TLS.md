---
title: 理解SSL/TLS
tags:
  - 安全性
  - 网络
date: 2021-10-06 16:18:11
---


## 什么是SSL/TLS

SSL的全称是Secure Sockets Layer，TLS的全称是Transport Layer Security。由于SSL和TLS都属于网络加密通信协议，且TLS是SSL的后续版，网络上许多文章都将这两者合起来一起称呼为SSL/TLS。

在SSL/TLS协议出现之前，HTTP通信是不加密的，这样的明文传播就带来了三个问题：
1. 消息可能被窃听
2. 消息可能被篡改
3. 发送者可能是被冒充的

为了解决这三个问题，我们可以使用SSL/TLS协议对HTTP通信进行加密。基于SSL/TLS进行加密的HTTP协议被称作HTTPS（HTTP Over SSL)。

## TLS协议结构
TLS协议的主要目标是保证隐私和两个应用程序之间的通讯数据完整性。该协议由两层组成：
- The TLS Record Protocol Layer（TLS记录协议层）：TLS记录协议层建立于可靠的传输上，比如TCP。这个协议层会为高层协议提供数据封装、压缩和加密等功能。
- The TLS Handshake Protocol Layer（TLS握手协议层）：TLS握手协议层建立于TLS记录协议层上。该层包含了三个子协议：握手协议（HandShake Protocol）、密码参数修改协议（Change Cipher Spec Protocol）和告警协议（Alert Protocol）。这些协议用于客户端与服务器数据传输之前进行身份认证，协商加密算法与交换加密密匙等等。

{% asset_img Transport_Layer_Security.svg Transport-Layer-Security %}

#### TLS记录协议层
TLS记录协议层包含了TLS记录协议。每一层消息包含了数据长度、数据描述和内容信息。对于需要进行传输的信息，记录协议会对消息进行分片，选择性地压缩数据，为每个片段增加MAC值，加密后进行传输。对于接收到的数据，记录协议会进行解密、验证、解压、重组，然后交付给更高级别的客户。

#### TLS握手协议层
该层包含了三个子协议，其中最重要也最复杂的是TLS握手协议。
- Change Cipher Spec Protocol（密码参数修改协议）：用于告知加密策略的变化。该协议只包含了一个ChangeCipherSpec消息，用于告知接收方后续的记录都会被新协商的加密算法和密钥进行保护和传输。
- Alert Protocol（告警协议）：用来允许一方向另一方报告告警信息。消息中包含告警的严重级别和描述。
- HandShake Protocol（握手协议）：客户端和服务器通过握手协议建立一个会话。握手协议包括客户端与服务器之间的一系列消息。该协议允许服务器和客户机相互验证，协商加密和MAC算法以及保密密钥，用来保护在SSL记录中发送的数据。握手协议包含了四个阶段，这四个阶段就是建立TLS通讯的核心。

## TLS握手过程

{% asset_img handshake.png handshake %}

- 第一阶段：确认安全参数。这个阶段，客户端和服务器会相互交换协商数据传输中要用到的相关安全参数。该阶段结束的时候，试图建立连接的算法将会对以下内容达成一致：
  - SSL/TLS版本
  - 加密算法
  - 压缩方法
  - 有关密钥生成的两个随机数：客户随机数和服务器随机数
  - 会话ID
- 第二阶段：在这个阶段，客户端是所有消息的接收方。服务器会将加密所需的证书发给客户端，该证书中将含括加密所需的公钥。同时服务器会对客户端进行身份验证。
- 第三阶段：在这个阶段，服务器是所有消息的接收方。客户端可以对收到的证书进行验证，如果证书不是可信机构颁布、或者证书中的域名与实际域名不一致、或者证书已经过期，就会向访问者显示一个警告，由其选择是否还要继续通信。通过再第一阶段收到的服务器随机数，根据不同的加密算法，客户端会算出一个pre-master密匙（Pre-master secret），使用数字证书中的公钥加密并发送给服务器。
- 第四阶段：服务器收到pre-master密匙后通过自己的私钥解密且计算生成本次会话所用的“会话密钥（session key）”，建立安全连接，通过发送一个ChangeCipherSpec告知客户端当前连接已经切换到协商过的加密套件状态，准备使用加密套件和session key加密数据了。


## 5. 参考资料
1. [SSL/TLS协议运行机制的概述](https://www.ruanyifeng.com/blog/2014/02/ssl_tls.html) 
2. [What is a TLS handshake?](https://www.cloudflare.com/zh-cn/learning/ssl/what-happens-in-a-tls-handshake/) 
3. [The Transport Layer Security (TLS) Protocol - Version 1.2](https://datatracker.ietf.org/doc/html/rfc5246) 
4. [Transport Layer Security (TLS)](https://hpbn.co/transport-layer-security-tls/) 
5. [SSL详解](https://blog.csdn.net/shipfsh_sh/article/details/80419994) 
6. [SSL协议原理](https://cloud.tencent.com/developer/article/1814069) 
7. [SSL/TLS协议详解](https://cshihong.github.io/2019/05/09/SSL%E5%8D%8F%E8%AE%AE%E8%AF%A6%E8%A7%A3/) 
8. [图解SSL/TLS协议](https://www.ruanyifeng.com/blog/2014/09/illustration-ssl.html) 