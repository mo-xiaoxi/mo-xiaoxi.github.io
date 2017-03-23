---
title: Tcpreplay
time: 2016.01.25 17:47:00
layout: post
catalog: true
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


1) tcpprep 用来区分客户端和服务器,tcprewrite 用来改写数据包,tcpreplay 负责发送数据包。因为数据包中的内容都是双向的(客户端->服务器,服务器->客户端),tcprewrite 改写数据, tcpreplay 发送数据包时都应该区分方向(即区分客户端和服务器),因此这两个工具一般是工作在 tcpprep 的基础上的。2)tcpreplay 可以发送任意 pcap 数据包,如果它想改变发送内容,就必须先用 tcprewrite 来改写数 据包,然后再发送改写后的数据包。3)tcpreplay-edit 是 tcprewrite 和 tcpreplay 的结合,其主要好处是不用生成改写后的包文件。
**指令风格：**
指令的选项分简短的和完整两种形式。
如查询版本的简短选项是“-V”,完整形式则是“--version”。
有的选项带有参数,而且两种选项的输入参数的形式不同。
如以缓存文件选项为例,对于简短选项, 形式是“-c pathname”;对于完整选项,形式是“--cachfile=pathname”。
pathname 是缓存文件的路径。

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
tcpprep [-flag [value]]... [--opt-name [value]]...
-a string,--auto=string自动分离模式,将自动分离出网络流量的 Client 端与 Server 端,记录在生成的 cache文件里。该模式将分别定义了 Client 与 Server 行为。 
Client 行为:- 发送 TCP SYN 数据包给另一台主机 
- 发送一个 DNS 请求- 收到一个 ICMP 端口不可达Server 行为:- 发送 TCP SYN/ACK 数据包给另一台主机- 发送一个 DNS 应答- 发送一个 ICMP 端口不可达
  如果数据包里的一个 IP 既有 Client 行为也有 Server 行为,自动分离模式将通过 Client Connections/Server Connections 的比值确定该 IP 是客户端,还是服务器端,若比值大于 2.0(默认值, 可以通过-R,--ratio 更改),则认为全部为客户端,否则认为全部为服务器端。自动分离模式根据处理 不符合 Client 与 Server 行为的数据包方式不同,分为四种方式,分别为 bridge、router、client 和 server:
  -a bridge,--auto=bridge	对于不符合 Client 与 Server 行为的数据包,该模式将输出分离失败信息并退出。	-a router,--auto=router		对于不符合 Client 与 Server 行为的数据包,该模式将未分离的 IP 所属网段与 Client 和 Server 端	进行对比,根据靠近原则将其划入最靠近的一端。	-a client,--auto=client	对于不符合 Client 与 Server 行为的数据包,该模式将其全部划入 Client 端。	-a server,--auto=server	对于不符合 Client 与 Server 行为的数据包,该模式将其全部划入 Server 端。	-a first,--auto=first 第一自动分离模式是一种特殊的自动分离模式,该模式是根据第一次看到的 IP 头 SRC 与 DST 信息来	区分客户端与服务器端,若一个 IP 第一次出现在 SRC 则认为是客户端,若第一次出现在 DST 则认为是服 务器端。若所抓包是完整的包,建议采用-a first 模式,若不是完整的包,建议不要采用-a first 模式, 而是根据坏境与需要采用上面四种模式。    -c string,--cidr=string	CIDR 分离模式,该模式将指定服务器网段并与源 IP 进行比较来分离客户端与服务器端。如: --cidr=192.168.0.0/16,172.16.0.0/12,10.0.0.0/8 该模式将认为数据包的源 IP 若在该三个网段,则认 为是服务器端,否则为客户端。    -r string,--regex=string 	正则分离模式,该模式将指定服务器网段并与源 IP 进行匹配来分离客户端与服务器端。如:tcpprep --regex="(10|20)\..*"该模式将认为所有的以 10.或 20.开头的源 IP 为服务器端。	-p,--port	端口分离模式,该模式默认认为所有目的端口<1024 的,将被视为客户端->服务器的包,否则视为服 务器->客户端的包。可以使用参数--services 自定义。	-e string,--mac—string	MAC 分离模式,该模式将指定服务器 MAC 地址并与源 MAC 进行比较来分离客户端与服务器端,匹配则 认为为服务器端,不匹配则认为为客户端。	-x,--include=string 仅仅处理匹配指定规则的数据包,不匹配的数据包不处理。 -X,--exclude=string 匹配指定规则的数据包不进行处理,处理不匹配的数据包。若	S:|D:|B:|E:10.0.0.0/8,192.168.0.0/16(其中 S=SIP,D=DIP,B=Both SIP and DIP,E=either SIP or DIP), P:1-5,9,15,72(编号),F:“tcp port 22”
  举例:[root@localhost src]# tcpprep -p -o ceshiftp.cache-i ftp.pcap

---

### 4.1.2 命令行解释	-a, --auto=str -c, --cidr=str 一般情况下都需要的参数,表示按模式自动分离通信流量生成 cache文件	-r, --regex=str   可选参数, 表示分离流量时采用 CIDR(无类别域间路由选择,是一个在 internet 上创建附加地址的方法,这些地址提供给服务提供商,再由服务提供商分配给客户。 CIDR 将路由集中起来,使一个 IP 地址代表主要骨干提供商提供的几千个 IP 地址, 从而减轻 internet 路由器的负担)	-p, --port  可选参数,表示使用 regex 模式分离通信流量,有点类似于 CIDR 模式,但是它匹 配的是服务器源 IP 可选参数,基于目的端口来分离通信流量,它区分的依据是任何 0-1023 端口都是 服务器的端发出的报文,其他的端口都是客户端发出的报文	-e,--mac=str  表示基于服务器源 MAC 地址分离通信流量	-C 可选参数,表示在 cache文件中嵌入注释内容,用于注释说明 cache文件的内容。使用位置不要放在最后,生成 cache后可以使用-P 查看写入的内容。	-x 重要可选参数,表示按照参数定义的需求来定义发送报文	-X 可选参数,是-x 的取反	-o 生成 cache文件必带参数,后跟 cache后缀的文件名,表示这个输出的 cache文件以这个文件名命名	-i 生成 cache文件必带参数,后跟 pcap 文件名,表示这个 pcap 文件需要处理	-P 可选参数,表示查看 cache文件的内容	-I 表示打印 cache文件的基本信息	-S 表示打印 cache文件的统计信息	-s 表示从服务器端口下载服务文件	-N 表示从服务器端口发送无 IP 流量	-R 可选参数,一个比例值。服务器端发起的连接数和客户端发起的连接数的比例,值大于 2 就视为服务器端	-m 可选参数,在选用 router 模式时使用,表示最小掩码,掩码默认是 30	-M 可选参数,在选用 router 模式时使用,表示最大掩码,默认是 8	-v 可选参数,显示 tcpprep 生成 cache文件的处理过程。信息的随时大于	-A 可选参数,在实验 tcpdump 风格打印输出信息时,同时再调用 tcpdump 中的参数 -h less help	-H help	-! more help### 4.1.3 Tcpprep用法举例
####  4.1.3.1根据报文源 IP 确定 client/server 报文	#tcpprep 的用法举例, 根据源 IP:	|$ tcpprep -c 172.22.64.2/24 -i mgcp.pcap -o mgcp.cache	上面的命令指定所有源 IP 为 172.22.64.2/24 的包, 都将从主网卡发出, 其它的从次网卡发出.	 输入文件是 mgcp.pcap, 输出文件为 mgcp.cache
#### 4.1.3.2 使用自动模式确定 client/server 报文	#tcpprep 的用法举例, 自动模式:	|$ tcpprep -a client -i mgcp.pcap -o mgcp.cach	上面的命令采用自动/client 模式指定分包模式. 自动模式这里按我的理解解释一下:	tcpprep 在自动模式下认为有下面行为的 IP 为 client 端:1、发 TCP SYN 包的一方,2、发 DNS 包 的一方,3、 收入到 ICMP-Port Unreachable 的一方; 认为有下面行为的一方为 server 端: 1、发 TCP Syn/Ack 的一方, 2、发 DNS 应答的一方,3.发 ICMP-Port Unreachable 的一方.而被认定为 server 的那 一方发的那些包, 将从主网卡发出, 被认定为client的包则从次网卡发出. 而自动/client模式将所有没有认 出的包都归为 client, 同理自动/server 模式将没有认出的包都归为 server.这种模式貌似不如按 IP 地址分类的方式好用.

---

### 4.2.1 Tcpwrite基本介绍 简单地说, tcprewrite 就是改写 pcap 包里的报文头部, 包括 2 层, 3 层, 4 层,即 MAC 地址、IP 地址和 PORT等。tcpreplay只保证能把包送出去, 至于包真正能到达的地址, 我认为还是根据原来的包的IP和mac, 如果是在同一网段,可能需要把 mac 地址改成测试设备的 mac,如果需要经过网关, 就得将 IP 地址改为 测试设备的 IP 以及端口号。### 4.2.1 参数简介
	$ tcprewrite 		--enet-smac=host_src_mac,client_src_mac \						--enet-dmac=host_dst_mac, client_dst_mac \ 						--endpoints=host_dst_ip:client_dst_ip \						 --portmap=old_port1:new_port1,old_port2, new_port2 \ 						 -i input.pcap -c input.cache-o out.pcap	--enet-smac=server_src_mac,client_src_mac 声明服务器源 MAC 地址,客户端源 MAC 地址 
	--enet-dmac=server_dst_mac, client_dst_mac 声明服务器端目的 MAC 地址,客户端目的 MAC 地址 
	--endpoints=server_src_ip:client_src_ip 声明服务器端源 IP 地址,客户端源 IP 地址 
	--seed=number按一定算法根据 number 的数值生成服务器端源 IP 地址,客户端源 IP 地址	 --pmap=old_ip_segment1:new_ip_segment1 旧的 IP 段改为新的 IP 段,不区分源 IP 段和目的 IP 段 
	 --srcipmap=old_src_ip_segment1:new_src_ip_segment1 旧的源 IP 段改为新的源 IP 段	--dstipmap=old_dst_ip_segment1:new_dst_ip_segment1  旧的目的 IP 段改为新的目地 IP 段	--portmap=old_port1:new_port1,old_port2, new_port2   旧的端口 port1 改为新的 port1,旧的端口 port2 改为新的端口 port2	-i input.pcap -c input.cache -o out.pcap  input.pcap 通过 tcpprep 生成的 input.cache来改写包文件内容,生成新的包文件 out.pcap### 4.2.2 命令行解释	
	-r 重写 tcp/udp 端口	-s 随机改写 IP 地址	-N 通过伪NAT重写ip地址	-e 在最后 2 个点之间重写 ip 地址	-b 不重写广播/多播 IP 地址	-C 强制重新计算 TP/RCP/UDP 校验和 -m 指定 MTU	-E 删除以太网最后一帧的校验和(删除最后 2 个字节)	-F 填充或截取包的数据以匹配包头长度	-d 输出调试信息(0-5,默认 0) 	-o 输出 pcap 文件	-I 处理输出 pcap 文件	-c 使用 tcpprep cache文件分离流量	-v tcpdump 风格打印对应信息	-A 可选参数,在使用 tcpdump 风格输出信息时,同时再调用 tcpdump 中的参数 -V 显示版本号### 4.2.3 用法举例	
	$ tcprewrite --enet-smac=11:22:22:22:22:22,22:22:22:22:22:22 
				 --enet-dmac=FF:FF:FF:FF:FF:FF				 --endpoints=192.168.0.1:192.168.0.11 
				 --portmap=5070:5061,9060:5060				 -i success.pcap -o out.pcap -c success.cach该命令将修改后的包, 主机端的二层, 三层, 四层头分别为: 11:22:22:22:22:22,192.168.0.1, 5061, 客户端包的二层, 三层, 四层头分别为: 22:22:22:22:22:22,192.168.0.11, 5060。		$ tcprewrite --seed=6 -i success.pcap -o out.pcap -c success.cache

该命令将根据数字 6 按特征算法生成一个新的 server_ip 和 client_ip。	
		$ tcprewrite --pmap=192.168.0.0/16:10.77.0.0/16,172.16.0.0/12:10.1.0.0/24 -i success.pcap -o out.pcap -c success.cache	
该命令将把主机端和客户端 192.168.0.0/16、172.16.0.0/12 网段的 ip 地址分别改为 10.77.0.0/16、10.1.0.0/24。	$ tcprewrite --srcipmap=192.168.0.0/16:10.77.0.0/16 -i success.pcap -o out.pcap -c success.cache
该命令将把主机端和客户端的源 ip 为 192.168.0.0/16 网段改为 10.77.0.0/16 网段。
	 $ tcprewrite --dstipmap=172.16.0.0/12:10.1.0.0/24 -i success.pcap -o out.pcap -c success.cache
该命令将把主机端和客户端的目的 ip 为 192.168.0.0/16 网段改为 10.77.0.0/16 网段。---
### 4.3.1 Tcpreplay介绍 Tcpreplay 根据 cache文件来回放 pcap 文件中的数据。### 4.3.2 命令行解释	-q 安静模式	-a 精确的时间(使用高速 cpu 发包)	-d 输出调试信息(0-5,默认 0)	-v 可选参数,每发送一个报文都以 tcpdump 风格打印对应信息	-A 可选参数,在使用 tcpdump 风格打印输出信息时,同时再调用 tcpdump 中的参数 -C 把报文存在内部缓存中	-c 双网卡回放报文必选参赛,后跟文件名	-I 双网卡回放报文必选参数,指定主接口	-I 双网卡回放报文必选参数,指定从借口	-N 获得网络接口和出口	-l 可选参数,指定循环次数	-S 制定包长度	-L 限制发包数量	-m 可选参数,指定一个倍数值,比默认发送速度快多少倍发送报文	-p 可选参数,指定每秒发送报文的个数。制定该参数,其他速率相关的参数被忽略,打印信息不会有速率和每秒发送报文的统计 	-M 以 Mbps(兆字节每秒)发送报文 	-t 以最快的速度回放报文	-o 一次回放一个报文	-P 可选参数,表示在输出信息中打印 PID 信息,用于单用户和单账户模式下暂停和重启程序	-V 显示版本号

### 4.3.3 用法举例	
	$tcpreplay --intf1=eth1 --intf2=eth0 –t --cachfile=cach_test.cachehttp_rewrite_pcap--intf1=eth1 是指主接口是 eth1,服务器->客户端的数据包通过这个接口发送。服务器和客户端的 区分是从 tcpprep 的处理结果 cach_test.cache中得到的。--intf2=eth0 是指从接口是 eth0,客户端->服务器的数据包通过这个接口发送。--cachfile=cach_test_cach是指tcpreplay用tcpprep 上步的处理结果--cach_test.cach来区分方 向。http_rewrite.pcap 是指 tcpreplay 发送的是来自 http_rewrite.pcap 这个文件中的数据包。 
如[root@localhost src]#tcpreplay –i eth1 –I eth0 –c ceshiftp.cacheceshiftp.pcap 

----
###  4.4.1 Tcpreplay-edit 的介绍tcpreplay-edit 实时修改包数据并回放,它是将 tcprewrite 和 tcpreplay 用一条命令实现。其好处 是修改包数据不会新生成 pcap 文件。如果是需要不断的改写一个包文件并回放建议使用 tcpreplay-edit, 如果是需要一次改写一个包文件并多次回放建议使用 tcprewrite 和 tcpreplay 的结合,这样具有更好的 回放速率。### 4.4.2 用法举例编写脚本,不断改写包文件的 IP 地址并回放: 
	for i in {1..255}	do	tcpreplay-edit --endpoints=1.1.2.$i:1.1.1.2 -t -i eth2 -I eth1 -c edit.cacheedit.pcap	done 通过 tcpreplay 来修改、转发通信流量需要考虑的一共需要考虑以下 3 点: 
 1.  确定哪有数据包是从客户端到服务器端的,哪有是从服务器端到客户端的 
      2.  确定新的 MAC、IP、Port 3. 确定回放速率、循环次数、执行方式根据以上 3 点按 3 步来完成操作:第一步:用 tcpreplay 分离源/目的端口的流量tcpprep --port --cachfile=example.cache--pcap=example.pcap这种情况下,认为所有目的端口<1024 的,将被视为客户端->服务器的包,否则视为服务器->客户端的包。该信息被存储在 tcpprp 的一个名叫 example.cache的文件夹中 
          -port 根据端口号区分数据包的流向
          --cachfile 指定输出的 cache文件的名字--pcap 指定要处理的数据包文件tcpprep 支持许多的其他的模式,分离端口模式是其中的一种第二步:使用 tcprewrite 更改 ip 地址到本地网络:$ tcprewrite --endpoints=172.16.0.1:172.16.5.35 --cachfile=example.cache --infile=example.pcap --outfile=new.pcap 
          这个例子里,我们想要所有的流量来自于 172.16.0.1 和 172.16.5.35。我们想要一个 IP 是“客户端”,一个 IP 是“服务器端”。
          --endpoints 指定数据包的 client、server 端的 ip 地址 --cachfie 上一步预处理的输出文件--infile 输入 pcap 文件--outfile 改写后的 pcap 文件
          第三步:用 tcpreplay,发送流量通过服务提供商
          tcpreplay --intf1=eth0 --intf2=eth1 --cachfile=example.cachenew.pcap因为我们要分离 2 个接口(eth0 和 eth1)之间的通信,我们使用第一步中创建的 cache文件,第二步 中创建的 new.pacp。然后使用 tcpreplay 重发数据包 
          --intf1--intf2---
## 5.演示使用Tcprelay重放攻击

1.	前期准备：首先，在Kali上添加一个网卡，便于测试。（虚拟机需处于关机状态）
    ![image](https://momomoxiaoxi.com/img/post/Tcpreplay/1.png)

然后，分别在OWASP虚拟机和SEEDUbuntu添加两个python测试文件：**OWASP：**
Root@owaspbwa:~# vim udp_s.py		import socket	# A UDP server	# Set up a UDP server	UDPSock = socket.socket(socket.AF_INET,socket.SOCK_DGRAM)	# Listen on port 21567	# (to all IP addresses on this system)	listen_addr = ("",1023)	UDPSock.bind(listen_addr)	# Report on all data packets received and	# where they came from in each case (as this is	# UDP, each may be from a different source and it's	# up to the server to sort this out!)	while True:            data,addr = UDPSock.recvfrom(1024)            print data.strip(),addr**SEEDUbuntu：**seed@ubuntu:~$ vim udp_c.py	import socket	# This is an example of a UDP client - it creates	# a socket and sends data through it	# create the UDP socket	UDPSock = socket.socket(socket.AF_INET,socket.SOCK_DGRAM)	data = " Hacked by Kali\n"	# Simply set up a target address and port ...	addr = ("192.168.1.76",1023)	# ... and send data out to it!	UDPSock.sendto(data,addr)实验流程说明:Kali虚拟机开启wireshark，监听整个网络，抓取网络信息。OWASP通过python代码（udp_s.py）运行UDP服务端，监听1023端口。SEEDUbuntu运行python代码（udp_c.py）运行UDP客户端，连接到OWASP虚拟机，并发送信息，因此OWASP显示“Hacked by Kali”信息。Kali虚拟机再依据wireshark获取的数据包，对OWASP虚拟机发起重放攻击。若攻击成功，则OWASP虚拟机，会再一次显示“Hacked by Kali”的信息。2.	在Kali开启wireshark抓包![image](https://momomoxiaoxi.com/img/post/Tcpreplay/2.png)2.	在OWASP节点，运行udp_s.py代码：	root@owaspbwa:~# python udp_s.py3.	在SEED节点，运行udp_c.py代码：	seed@ubuntu:~$ python udp_c.py可以分别在OWASP与Kali节点上，看到下列信息：![image](https://momomoxiaoxi.com/img/post/Tcpreplay/3.png)
OWASP节点信息![image](https://momomoxiaoxi.com/img/post/Tcpreplay/4.png)Kali节点信息4.	在Kali节点，保存抓取的包信息，保存为UDP.pcap：![image](https://momomoxiaoxi.com/img/post/Tcpreplay/5.png)保存信息5.	使用tcpprep分离源／目的地址流量，然后使用tcpreplay进行重放攻击：
​	
		root@kali:~# tcpprep -a client -i UDP.pcap -o UDP.cache		（说明：采用自动/client 模式指定分包模式. 自动/client模式将所有没有认出的包都归为 client, 同理自动/server 模式将没有认出的包都归为 server. UDP.pcap是输入文件，UDP.cache是输出文件）		root@kali:~# tcpreplay -p 10 -c UDP.cache -i eth0 -I eth1 UDP.pcap 		sending out eth0 eth1		processing file: UDP.pcap		Actual: 3 packets (180 bytes) sent in 0.20 seconds.			Rated: 900.0 bps, 0.01 Mbps, 15.00 pps		Statistics for network device: eth0		Attempted packets:         0		Successful packets:        0		Failed packets:            0		Retried packets (ENOBUFS): 0		Retried packets (EAGAIN):  0		Statistics for network device: eth1			Attempted packets:         3			Successful packets:        3			Failed packets:            0		Retried packets (ENOBUFS): 0		Retried packets (EAGAIN):  0说明:-p 表示指定每秒发送速度 –c –i –I 均是双网卡回放报文必选参数，分别跟输入文件名 主接口 次接口。导入UDP.cache主要用于区分方向，服务器->客户端的数据包通过主接口发送，客户端->服务器的数据包通过次接口发送。而tcpreplay重放的是来自UDP.pcap中的数据包。
6.	可以在OWASP节点，明显得看到多了一条显示信息（Hacked by Kali），表明重放攻击成功
    ![image](https://momomoxiaoxi.com/img/post/Tcpreplay/6.png)
---
## 6. 参考资料
http://tcpreplay.synfin.nethttps://en.wikipedia.org/wiki/Replay_attackhttps://cnodejs.org/topic/557c354d16839d2d539362b6https://github.com/appneta/tcpreplay