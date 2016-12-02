---
title: DNS、DNSSEC、QUIC协议学习
time: 2016.12.02 10:00:00
layout: post
catalog: true
tags:
- Security
- DNS

excerpt: 这里对DNS、SNSSEC、QUIC协议进行一个简单的学习与总结。


---



## 协议学习

#### 1. DNS

   DNS最基础的作用就是实现域名到ip地址的映射工作。后期，经过扩展它还能提供主机别名、邮件服务器别名、负载分配等等功能。

   DNS一般运行在UDP之上，使用53端口。

   DNS类型一般分为下列几种：

|  类型  |  助记符  |          说明           |
| :--: | :---: | :-------------------: |
|  1   |   A   |        IPv4地址         |
|  2   |  NS   |         名字服务器         |
|  5   | CNAME |   规范名称定义主机的正式名字的别名    |
|  6   |  SOA  |     开始授权标记一个区的开始      |
|  11  |  WKS  |    熟知服务定义主机提供的网络服务    |
|  12  |  PTR  |     指针把IP地址转化为域名      |
|  13  | HINFO | 主机信息给出主机使用的硬件和操作系统的表述 |
|  15  |  MX   |  邮件交换把邮件改变路由送到邮件服务器   |
|  MX  | AAAA  |        IPv6地址         |
| 252  | AXFR  |       传送整个区的请求        |
| 255  |  ANY  |       对所有记录的请求        |

   更具体的信息请参考：

   [DNS协议](http://blog.csdn.net/zhaqiwen/article/details/18048791)

   [域名解析中A记录、CNAME、MX记录、NS记录的区别和联系](http://blog.csdn.net/crazw/article/details/8986581)

    [DNS解析过程详解](http://blog.csdn.net/crazw/article/details/8986504)

#### 1. DNSSEC

   > **域名系统安全扩展**（英语：Domain Name System Security Extensions，缩写为DNSSEC）是[Internet工程任务组](https://zh.wikipedia.org/wiki/IETF) （[IETF](https://zh.wikipedia.org/wiki/IETF)）的对确保由[域名系统](https://zh.wikipedia.org/wiki/%E5%9F%9F%E5%90%8D%E7%B3%BB%E7%BB%9F) （[DNS](https://zh.wikipedia.org/wiki/DNS)）中提供的关于互联网协议 （IP）网络使用特定类型的信息规格套件。它是对DNS提供给DNS客户端（解析器）的DNS数据来源进行认证，并验证不存在性和校验数据完整性验证，但不提供或机密性和可用性——wiki

   DNSSEC协议可帮助域名系统（DNS）数据中的数字签名来验证数据的来源以及检查其在互联网传输过程中的完整性。同时，基于DNS的域名实体认证（DANE）会通过使用DNSSEC来绑定传输层安全（TLS）到DNS，以确保证书来自特定的证书颁发机构。

   具体推荐参考学习：

   1. （推荐！！！很清晰）段老师的博客：[DNSSEC 原理、配置与布署简介](http://netsec.ccert.edu.cn/duanhx/?p=1479)
   2. [DNSSSEC验证的加密技术](http://www.jiamisoft.com/blog/18245-qinxi.html)

 #### 3. QUIC协议

   Quick、UDP、Internet、Connections

   QUIC的主要特点包括：**具有SPDY(SPDY是谷歌研制的提升HTTP速度的协议，是HTTP/2.0的基础)所有的优点；0-RTT连接；减少丢包；前向纠错，减少重传时延；自适应拥塞控制， 减少重新连接；相当于TLS加密。**

   QUIC在协议中的位置：

![3](/img/post/DNSSEC/1.png)


- 为什么需要QUIC协议：

    因为SPDY与TCP本身有一些固有的缺陷。如基于一条TCP连接的SPDY复用连接会面临这样的情况：当有丢包发生时，所有连接都将阻塞，这是由TCP的拥塞控制特性决定的。丢包必须恢复，而恢复过程中，或早或晚，滑动窗口总有停等的时刻，耗费一个RTT。在广域网上，一个RTT相当于50-100ms。相比较而言，当x条并行HTTP连接中，有一条丢包，只会阻塞一条。所以，我们需要QUIC协议来加快网络访问速度。


- 重传与恢复（这是QUIC最有特色的一个方面）

  丢包恢复一共有两种方法：前向纠错（FEC）和重传。前向纠错可以减少重传，但需要在保重添加冗余信息，用XOR实现。如果前向纠错不能回复包，才启用重传，重传的不是旧包，而是重新构造的包。

  前向纠错（FEC）：FEC采用简单异或的方式。每次发送一组数据，包括若干个数据包后，并对这些数据包依次作异或运算，最后的结果作为一个FEC包再发送出去。接收方收到一组数据后，根据数据包和FEC包即可以进行校验和纠错。

  对于某些重要的数据包，如初始密钥协商时的数据包，在建立连接时非常重要，如果这类包丢失会阻塞整体数据流。QUIC对于这一类数据包在确认发生丢失前就会尝试重传，通常是等待较短的时间(如20ms)没收到确认后就马上再次发送。这样在网络中会有若干个相同的包同时传输，只要有一个能成功抵达就完成了连接，这样降低了丢包率。接收方对于关键数据包的多次发送和普通数据包的超时重传，都采用相同的重复包处理机制

  QUIC在拥塞避免算法的基础上还加入了心跳包，用于减少丢包率

- 安全性

     QUIC对每个散装的UDP包都进行了加密和认证的保护，并且避免使用前向依赖的处理方法(如CBC模式)，这样每个UDP包可以独立地根据IV进行加密或认证处理。
    QUIC采用了两级密钥机制：初始密钥和会话密钥。初次连接时不加密，并协商初始密钥。初始密钥协商完毕后会马上再协商会话密钥，这样可以保证密钥的前向安全性，之后可以在通信的过程中就实现对密钥的更新。接收方意识到有新的密钥要更新时，会尝试用新旧两种密钥对数据进行解密，直到成功才会正式更新密钥，否则会一直保留旧密钥有效。

- o-RTT握手过程
    QUIC握手的过程是需要一次数据交互，0-RTT时延即可完成握手过程中的密钥协商，比TLS相比效率提高了5倍，且具有更高的安全性。
    QUIC在握手过程中使用Diffie-Hellman算法协商初始密钥，初始密钥依赖于服务器存储的一组配置参数，该参数会周期性的更新。初始密钥协商成功后，服务器会提供一个临时随机数，双方根据这个数再生成会话密钥。

   具体握手过程如下：
   (1) 客户端判断本地是否已有服务器的全部配置参数，如果有则直接跳转到(5)，否则继续
   (2) 客户端向服务器发送inchoate client hello(CHLO)消息，请求服务器传输配置参数
   (3) 服务器收到CHLO，回复rejection(REJ)消息，其中包含服务器的部分配置参数
   (4) 客户端收到REJ，提取并存储服务器配置参数，跳回到(1)
   (5) 客户端向服务器发送full client hello消息，开始正式握手，消息中包括客户端选择的公开数。此时客户端根据获取的服务器配置参数和自己选择的公开数，可以计算出初始密钥。
   (6) 服务器收到full client hello，如果不同意连接就回复REJ，同(3)；如果同意连接，根据客户端的公开数计算出初始密钥，回复server hello(SHLO)消息，SHLO用初始密钥加密，并且其中包含服务器选择的一个临时公开数。
   (7) 客户端收到服务器的回复，如果是REJ则情况同(4)；如果是SHLO，则尝试用初始密钥解密，提取出临时公开数
   (8) 客户端和服务器根据临时公开数和初始密钥，各自基于SHA-256算法推导出会话密钥
   (9) 双方更换为使用会话密钥通信，初始密钥此时已无用，QUIC握手过程完毕。之后会话密钥更新的流程与以上过程类似，只是数据包中的某些字段略有不同。

![2](/img/post/DNSSEC/2.png)

​	[QUIC客户端源码](https://cs.chromium.org/chromium/src/net/quic/?q=quic&sq=package:chromium)

​	[QUIC服务端源码](https://cs.chromium.org/chromium/src/net/tools/quic/?q=quic&sq=package:chromium)

------

## 参考

1. [DNS隧道](http://blog.codingnow.com/2011/06/dns_tunnel.html)

2. [DNS协议](http://blog.csdn.net/zhaqiwen/article/details/18048791)

3. [域名解析中A记录、CNAME、MX记录、NS记录的区别和联系](http://blog.csdn.net/crazw/article/details/8986581)

4. [DNS解析过程详解](http://blog.csdn.net/crazw/article/details/8986504)

5. [DNSSEC 原理、配置与布署简介](http://netsec.ccert.edu.cn/duanhx/?p=1479)

6. [DNSSSEC验证的加密技术](http://www.jiamisoft.com/blog/18245-qinxi.html)

7. [Google QUIC协议：从TCP到UDP的Web平台](http://www.infoq.com/cn/articles/quic-google-protocol-web-platform-from-tcp-to-udp)

8. [QUIC和TCP](http://blog.chinaunix.net/uid-28387257-id-4335291.html)

9. [QUICgoogle文档](https://docs.google.com/document/d/1RNHkx_VvKWyWg6Lr8SZ-saqsQx7rFV-ev2jRFUoVD34/edit#)

   ​