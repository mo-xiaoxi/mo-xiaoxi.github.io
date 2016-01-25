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
## 4.参数介绍
（1）tcpprep使用的参数：

Usage: # tcpprep [-a -n <mode> -N <type> | -c <cidr> | -p | -r <regex>]  -o <out> -i <in> <args>

-a  Split traffic in Auto Mode

一般情况下都需要该参数，表示按模式自动分离的通讯流量生成cache文件，表示自动分离采取的拓扑模式，来决定采取那种模式分离通讯流量的双方。

-c  CIDR1,CIDR2,...      Split traffic in CIDR Mode

可选参数，表示分离流量时采用CIDR（无类别域间路由选择）模式。格式：tcpprep  -ac10.10.0.0/24，表示把源地址匹配10.10.0.0/24网段的报文全部由主网卡发送，剩下的报文由从网卡发送出来，这里还有一点需要补充，就是tcpreplay在重放报文时对两个网卡的定义很明确，一个主网卡（primary interface），一个是从网卡（secondary interface），不同的模式，两块网卡的属性不一样。

-C <comment>           Embed comment in tcpprep cache file

可选参数，表示在cache文件中嵌入注释内容，可以用于注释说明cache文件的内容，注意使用时参数位置，不要放在最后，我测试时放在-o参数值的后面就报错，放到-i参数之前就可以。生成cache文件后使用-P可以查看写入的内容。（这个参数一般用不着）

-h     Help

显示帮助文件

-i <capfile>            Input capture file to process

生成cache文件的必带参数，后面紧跟pcap文件名，表示这个pcap文件需要处理。

-m <minmask>            Minimum mask length in Auto/Router mode

可选参数，在选用router模式时使用，表示最小掩码，默认是30（2个有效ip地址）。

-M <maxmask>            Maximum mask length in Auto/Router mode

可选参数，在选用router模式时使用，表示最大掩码，默认是8（1600万个ip地址）。

-n <auto mode>          Use specified algorithm in Auto Mode

生成cache文件的必带参数，后面紧跟模式名称，可选项有(bridge|router|client|server)，目前的版本只支持这4种模式。模式的选择很关键，例如在客户端使用ftp软件下载文件，那么你在客户端抓到的报文生成的pcap文件，那么就选用client模式，在服务器端抓到的报文生成的pcap文件就选用server模式。只有模式选对了，才能正确的分离流量从正确的接口发出正确的报文。注意：Server端的报文由主网卡发送出去，Client端的报文由从网卡发送出去。怎么确定主从网卡由tcpreplay的命令（-i –I两个参数）来决定。

-o <outputfile>         Output cache file name

生成cache文件的必带参数，后面紧跟cache文件名，表示这个输出的cache文件以这个名字命名。

-p        Split traffic based on destination port

可选参数，基于目的端口来分离通讯流量，它区分的依据是认为0-1023端口都是服务器的端发出的报文，其它的端口都是客户端发出的报文，具体的端口对应的/etc/services文件里的的内容。使用的格式：-p /etc/services，可以根据自己的需要来制作一个文件也可以。

-r <regex>              Split traffic in Regex Mode

可选参数，表示使用Regex模式分离通讯流量，有点类似于CIDR模式，但是它匹配的是服务器的源IP。man文件提示不能和-a、-c参数一起使用。

-R <ratio>              Specify a ratio to use in Auto Mode

   可选参数，一个比例值，这个比例值的意义是服务器端发起的连接数和客户端发起的连接数的比例，这个值大于2的话就视为server端。这个英文原意我也不是太肯定，大家可以参考一下原文：

The ratio of server connections to client connections  necessary to  be classified as a server in auto mode.  A system is classified as a server if [# server connections] >= ([# client connections] * [ratio]).  Default is: 2.0

-x <match>              Only send the packets specified

  重要的可选参数，表示按照参数定义的需求来定义发送报文。后面还有具体的参数，因为在我们的抓包过程中，可能会由于网络环境原因，抓到了许多我们不需要回放的报文，我们就可以根据这个参数决定我们需要回放哪些报文内容。具体的参数意思如下：

S:<CIDR1>,... - Src IP must match specified CIDR(s)

  在CIDR模式下必须匹配源IP，格式：-xS:100.1.1.0/24,10.10.10.0/26。多个用逗号隔开，参数个数没有试过，3个没有问题。

D:<CIDR1>,... - Dst IP must match specified CIDR(s)

在CIDR模式下必须匹配目的IP，格式同上。

B:<CIDR1>,... - Both src and dst addresses must match

必须同时匹配源和目的IP，格式同上。

E:<CIDR1>,... - Either src or dst address must match

匹配源或目的IP，格式同上。

Ex: -xP:1-5,9,15 would only send packets 1 through 5, 9 and 15.

根据参数后的参数值（报文编号）发送指定的报文。可以在ethereal中确认报文的编号，然后把需要的报文发送。可以用于排除ARP报文。

Tcpprep使用小结：在使用过程中，很多参数都没有用到，用的比较多的选项参数就-v、-P、-xB、-xP，一般都是client和server的模式，其它两种模式没有实验过，暂时还不知道怎么使用。

Tcpprep区分模式举例：

1）tcpprep -e 00:00:00:00:00:05 --include=P:1,2,3 -i test.pcap -o test.cache &（mac模式include）

2）tcpprep -e 00:1e:c9:4c:03:0a --exclude=P:1,2-5 -i test.pcap -o test.cache &（mac模式exclude）

3）tcpprep -a server --exclude=P:1,2-5 -i test.pcap -o test.cache &（auto模式）

4）tcpprep --cidr=192.168.0.0/16,10.0.0.0/8 --include=P:1,2-5 -i test.pcap -o test.cache & （CIDR）

5）tcpprep -p  --include=P:1,2-5 -i test.pcap -o test.cache & （port模式）

（2）tcprewrite使用的参数

-r   rewrite TCP/UDP ports

重写TCP/UDP端口

-e  rewrite IP addresses to be between two endpoints

重写两个端点之间的IP地址

-b  skpi rewriting broadcast/multicast IPv4/IPv6 addresses

跳过重写广播/多播IPv4/IPv6地址

--enet-dmac    Override destination Ethernet MAC addresses

修改目的MAC地址

--enet-smac    Override source Ethernet MAC addresses

修改源端MAC地址

-i   Input pcap file to be processed

输入被处理的pcap文件

-o  out pcap file

输出pcap文件 （这里这个生成的文件我还没有弄清楚到底是什么作用）

-c   Split traffic via tcpprep cache file

发送的文件，即tcpprep处理生成的cache文件

-h   help

显示帮助

（tcprewrite 工具还没有真正的使用测试过，因此具体功能还要验证）

（3）tcpreplay使用的参数：

Usage: tcpreplay [args] <file(s)>

-c <cachefile>          Split traffic via cache file

 双网卡回放报文必选参数，后面紧跟cache文件名，该文件为tcpprep根据对应的pcap文件构造出来。

-F           Fix IP, TCP, UDP and ICMP checksums

   可选参数，在发送报文时，自动纠正错误的校验和。对测试DUT的校验和检验还是有用的。

-i <nic>                Primary interface to send traffic out of

 双网卡回放报文必选参数，指定主接口。

-I <nic>                Secondary interface to send traffic out of

  双网卡回放报文必选参数，指定从接口。

-L <limit>              Specify the maximum number of packets to send

   可选参数，指定最大的发包数量。可以在确认连接的调试时使用。

-x <multiple>           Set replay speed to given multiple

   可选参数，指定一个倍数值，就是必默认发送速率要快多少倍的速率发送报文。加大发送的速率后，对于DUT可能意味着有更多的并发连接和连接数，特别是对于BT报文的重放，因为连接的超时是固定的，如果速率增大的话，留在session表中的连接数量增大，还可以通过修改连接的超时时间来达到该目的。

-p <packetrate>         Set replay speed to given rate (packets/sec)

   可选参数，指定每秒发送报文的个数，指定该参数，其它速率相关的参数被忽略，最后的打印信息不会有速率和每秒发送报文的统计。

这是是主要的一些参数，从网上查找的一些资料跟具体的tcpreplay中的参数有很大出入，因此建议想要使用其他参数时，使用命令:# tcpreplay –h 查看帮助。


---
## 5.Tcprelay重放攻击
演示过程：利用wireshark抓取数据包后，tcprelay重放数据包，再次抓包分析。