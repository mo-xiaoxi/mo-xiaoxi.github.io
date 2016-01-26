---
title: Tcpreplay
time: 2016.01.25 17:47:00
layout: post
tags:
- Security
- Tcpreplay
excerpt: 主要介绍一些Tcpreplay工具的使用方法，与通过结合wireshark演示一个攻击重放的案例
    
---


#Tcpreplay

---
## 1. 简介
Tcpreplay是一系列工具的总称，包括tcpreplay、tcprewrite和tcpprep等工具，它可以用来在Unix系统或者linux系统上重放网络包。这些包是由tcpdump、ethereal和wireshark等软件抓取到的，即pcap格式的数据包。

简单的说, tcpreplay是一种pcap包的重放工具, 它可以将用ethreal, wireshark工具抓下来的包原样或经过任意修改后重放回去. 它允许你对报文做任意的修改(主要是指对2层, 3层, 4层报文头), 指定重放报文的速度等, 这样tcpreplay就可以用来复现抓包的情景以定位bug, 以极快的速度重放从而实现压力测试.


安装Tcpreplay包时，默认情况下是安装了下面这些工具的, 用于准备发包的cache, 重写报文等:

1. tcpprep: 这个工具的作用就是划分客户端和服务器，区分pcap数据包的流向，即划分那些包是client的，哪些包是server的，一会发包的时候client包从一个网卡发，另一个server的包可能从另一个网卡发。

2. tcprewrite：这个工具的作用就是来修改报文，主要修改2层，3层，4层报文头，即MAC地址，IP地址和PORT地址。

3. tcpreplay: 这是最终真正发包使用的工具，可以选择主网卡、从网卡、发包速度等。

4. tcpliveplay:使用新的tcp连接，来进行重放，数据来源是pcap文件

5. tcpreplay-edit：重放数据包编辑
 
 6. tcpbridge：通过tcpwrite桥接两个网络段
 
 7. tcpcapinfo:对pcap文件进行编码或调试

工具作用通俗解释：

 
1) tcpprep 用来区分客户端和服务器,tcprewrite 用来改写数据包,tcpreplay 负责发送数据包。因为数据包中的内容都是双向的(客户端->服务器,服务器->客户端),tcprewrite 改写数据, tcpreplay 发送数据包时都应该区分方向(即区分客户端和服务器),因此这两个工具一般是工作在 tcpprep 的基础上的。







---
## 2.安装环境

Tcpreplay要实现它的功能要用到其它一些库：

1）libpcap库：由于tcpreplay在使用过程中主要依赖于libpcap库，因此在安装tcpreplay之前，需要先安装libpcap，否则在安装tcpreplay的时候会提示你libpcap没有安装而安装失败。但如果希望libpcap能在linux上正常工作，则必须使内核支持"packet"协议，也即在编译内核时打开配置选项 CONFIG_PACKET(选项缺省为打开)。

Libpcap可以从以下链接下载：[link](http://www.tcpdump.org/)  可以用源码安装，比较简单。

2）tcpdump： 这个不是必须的，这个工具的主要作用就是来解码数据包，在linux系统下查看pcap文件内的数据包，也可以用它来进行抓包。在安装tcpreplay时可以选择安装它也可以不选择安装。（个人建议也一起安装，以防止在使用tcpreplay时发生其他错误。）

3）libnet库。非必需库。Tcpreplay也可用它来发送数据包，但由于libnet自身的bug比较多且已不再有人维护，Tcpreplay未来版本有可能取消对它的支持。（目前的版本还需要这个库的支持，因此建议也安装）

---
## 3.安装过程
这个网上有很多，大家参考下就行。

---
## 4.基本介绍
tcpprep 是一个在 tcprewrite 和 tcpreplay 之前使用的 pcap 文件的处理程序。使用 tcpprep 的目的 就是建立一个
cache文件,用于分离通信流量中的两方(通常叫做主要的/次要的或者 客户端/服务器), 为 tcprewrite 和 tcpreplay 处理
与发送报文做准备。如果你正打算在两块网卡上使用 tcpreplay 的话, 那么 tcpprep 就是用来决定每一个报文(packet)从哪
一个接口发出。通过使用这样一个分离的程序来建 立一个 cache文件,tcpreplay 就可以根据这个 cache文件通过自身的计算来
分离流量,高速率的发送报文。 cache文件的作用主要是加速报文的发送,cache文件中存放着 pcap 文件中每个帧的编号和时间戳
等信息, 以达到 tcpreplay 回放时可以更加快速的发送报文的目的。

---
### 4.1.1 Tcpprep参数介绍








---
### 4.1.2 命令行解释



---
### 4.2.1 Tcpwrite基本介绍







				 --enet-dmac=FF:FF:FF:FF:FF:FF
				 --portmap=5070:5061,9060:5060

该命令将根据数字 6 按特征算法生成一个新的 server_ip 和 client_ip。









如[root@localhost src]#tcpreplay –i eth1 –I eth0 –c ceshiftp.cacheceshiftp.pcap

----










演示过程：利用wireshark抓取数据包后，tcprelay重放数据包，再次抓包分析。
然后，分别在OWASP虚拟机和SEEDUbuntu添加两个python测试文件：