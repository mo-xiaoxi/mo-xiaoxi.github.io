---
title: HCTF2016 Web WP
time: 2016.10.29 10:00:00
layout: post
catalog: true
tags:
- Security
- CTF
- Web

excerpt: 上个周末，一起和大家打了HCTF比赛，比赛很有趣，质量也很高。前面三层难度适中，后面两层的题目就完全不会做了orz
---



# DNS tunnel+Socks代理配置

> 我们平时在渗透过程中，有时候需要使用DNS隧道来反弹shell。有时候，在平日上网的时候，需要绕过web认证。有时候，我们需要VPN翻越长城防火墙。。。。
>
> 所以，我打算配置一个DNS tunnel+socks代理来解决这些问题。

## 环境准备

1. 一台具有独立公网IP的服务器
2. 一个可控的域名（可配置DNS）
3. DNS tunnel工具===》这里采用iodine，[源码](https://github.com/yarrick/iodine)
4. Socks代理====》这里采用shadowsocks，[源码](https://github.com/shadowsocks/shadowsocks/tree/master)



## DNS解析配置

你需要在DNS解析配置中，添加两条记录。

一条是NS记录，a.yourdomain.com，纪录值是”b.yourdomain.com.”(不要忘了最后一个dot)

另一条是A记录，b.yourdomain.com，记录值是server端的公网IP（example: 123.456.789.123）

设置好了以后可以在[http://code.kryo.se/iodine/check-it/](http://code.kryo.se/iodine/check-it/)输入a.yourdomain.com测试是否配置成功。

如图：

![](../img/post/iodine/1.png)

此时，你在测试网站上看到的情景如下：

![](../img/post/iodine/2.png)

## 服务器配置iodine

> linux版本：[root@cloud ~]# cat /proc/version 
> Linux version 2.6.32-358.el6.x86_64 (mockbuild@c6b8.bsys.dev.centos.org) (gcc version 4.4.7 20120313 (Red Hat 4.4.7-3) (GCC) ) #1 SMP Fri Feb 22 00:31:26 UTC 2013

```Bash
# git clone https://github.com/yarrick/iodine.git
# cd iodine
# make
# make install
```

如果出现下列错误

```Bash
make[1]: Entering directory `/home/iodine/src'
OS is LINUX, arch is x86_64
CC client.c
client.c:29:18: 错误：zlib.h：没有那个文件或目录
client.c: 在函数‘read_dns_withq’中:
client.c:659: 警告：隐式声明函数‘uncompress’
client.c:659: 错误：‘Z_OK’未声明(在此函数内第一次使用)
client.c:659: 错误：(即使在一个函数内多次出现，每个未声明的标识符在其
client.c:659: 错误：所在的函数内也只报告一次。)
client.c: 在函数‘tunnel_tun’中:
client.c:770: 警告：隐式声明函数‘compress2’
client.c: 在函数‘tunnel_dns’中:
client.c:1013: 错误：‘Z_OK’未声明(在此函数内第一次使用)
make[1]: *** [client.o] 错误 1
make[1]: Leaving directory `/home/iodine/src'
make: *** [all] 错误 2
```

可以通过安装zlib，解决即可

```
[root@cloud iodine]# iodine
Usage: iodine [-v] [-h] [-f] [-r] [-u user] [-t chrootdir] [-d device] [-P password] [-m maxfragsize] [-M maxlen] [-T type] [-O enc] [-L 0|1] [-I sec] [-z context] [-F pidfile] [nameserver] topdomain
[root@cloud iodine]# iodined
Usage: iodined [-v] [-h] [-4] [-6] [-c] [-s] [-f] [-D] [-u user] [-t chrootdir] [-d device] [-m mtu] [-z context] [-l ipv4 listen address] [-L ipv6 listen address] [-p port] [-n external ip] [-b dnsport] [-P password] [-F pidfile] [-i max idle time] tunnel_ip[/netmask] topdomain
```

运行iodined

```
[root@cloud ~]# iodined -c -f -P t4st -4 192.168.99.1 xiaoxi.ilovefyy.top
Opened dns0
Setting IP of dns0 to 192.168.99.1
Setting MTU of dns0 to 1130
Opened IPv4 UDP socket
Listening to dns for domain xiaoxi.ilovefyy.top

```

此时，测试数据如下

![](../img/post/iodine/4.png)

## 客户端安装iodine

### macOS Sierra

```bash
# git@github.com:yarrick/iodine.git
# cd iodine
# sudo make
# sudo make install
```

然后，配置下环境

在~/.zshrc中配置如下：

```
# iodine
PATH="/usr/local/sbin:${PATH}"
export PATH
```

source ~/.zshrc

接下来就可以使用iodine了

然后安装tuntap即可。

这里，我下载了[源码]([http://ncu.dl.sourceforge.net/project/tuntaposx/tuntap/20150118/tuntap_20150118_src.tar.gz](http://ncu.dl.sourceforge.net/project/tuntaposx/tuntap/20150118/tuntap_20150118_src.tar.gz))

然后，编译安装

```
sudo make
sudo make install
```

>这里，我测试的时候不知道为什么一直会有段错误？求教？😢

安装完成：

![](../img/post/iodine/3.png)

下面采用kali的虚拟机进行测试：

![](../img/post/iodine/5.png)

连接成功。

互相ping一下：

![](../img/post/iodine/6.png)

![](../img/post/iodine/7.png)

正常连接



## 安装shadowsocks

### Server

Debian / Ubuntu:

```
apt-get install python-pip
pip install shadowsocks

```

CentOS:

```
yum install python-setuptools && easy_install pip
pip install shadowsocks
```

### Usage

```
ssserver -p 443 -k password -m aes-256-cfb

```

To run in the background:

```
sudo ssserver -p 443 -k password -m aes-256-cfb --user nobody -d start

```

To stop:

```
sudo ssserver -d stop

```

To check the log:

```
sudo less /var/log/shadowsocks.log
```

开启ssserver

![](../img/post/iodine/8.png)

### Client

大多数客户端从[ishadowsocks](http://www.ishadowsocks.net/)就可以获取

下面演示在MacOS Sierra上进行测试：

配置端口，进行连接



![](../img/post/iodine/10.png)

![](../img/post/iodine/9.png)



![](../img/post/iodine/12.png)

连接成功。

成功连接上google

![](../img/post/iodine/11.png)

ssh测试：

![](../img/post/iodine/13.png)

---



由于先前的iodine，我在mac上安装失败，所以我在这演示在kali上安装shadowsocks，并通过客户端Kali与服务端CentOS来进行最终的测试。即测试iodine+shadowsocks的通信。（这里比较蛋疼，发现在虚拟机中开启shadowsocks无法连接到远程的服务器上）

安装：

 ```
sudo apt-get install python-pip
sudo pip install shadowsocks
 ```

启动客户端很简单，可以参照如下

`sslocal -s server_ip -p server_port  -l 1080 -k password -t 600 -m aes-256-cfb`

-s表示服务IP, -p指的是服务端的端口，-l是本地端口默认是1080（可以替换成任何端口号，只需保证端口不冲突）, -k 是密码（要加""）, -t超时默认300,-m是加密方法默认aes-256-cfb，

可以简单的写为：`sslocal -s ip  -p  port -k "password" `   #用-s -p -k这三个参数就好，其他的默认将服务端的加密方法设为aes-256-cfb。然后就可以启动代理。

> 这里没办法连接。

所以，这里单独测试ssh功能，可以发现iodine之后，ssh是可以连接的。那么如果shadowsocks可以连接，那么我们就可以通过shadowsock进行通信了。





----

## 补充

由于前面使用mac进行客户端运行iodine时失败了，这里尝试使用dns2tcp

#### 服务器安装

```bash
wget http://www.hsc.fr/ressources/outils/dns2tcp/download/dns2tcp-0.5.2.tar.gz  
tar zxf dns2tcp-0.5.2.tar.gz  
cd dns2tcp-0.5.2  
./configure  
make & make install 
```

建立配置文件

```bash
[root@cloud dns2tcp-0.5.2]# touch /etc/dns2tcpd.conf
[root@cloud dns2tcp-0.5.2]# vim /etc/dns2tcpd.conf 
[root@cloud dns2tcp-0.5.2]# cat /etc/dns2tcpd.conf 
listen=118.99.13.97
port= 999
user = nobody
chroot = /tmp
domain = xiaoxi.ilovefyy.top
resources = ssh:127.0.0.1:22,socks:127.0.0.1,http:127.0.0.1:3128
```

最后的 resources 里面配置的是 DNS2TCP 供客户端使用的资源. 

作用是: 客户端在本地监听一个22的端口,传送数据到指定端口，DNS2TCP 客户端将数据用DNS协议传动到服务器端口，然后服务器将数据转发到对应的资源配置的端口中

运行dns2tcpd

```bash
[root@cloud dns2tcp-0.5.2]# dns2tcpd -f /etc/dns2tcpd.conf -F -d 2
16:23:05 : Debug options.c:97	Add resource ssh:127.0.0.1 port 22
16:23:05 : Debug options.c:97	Add resource http:127.0.0.1 port 3128
16:23:05 : Debug socket.c:55	Listening on 118.99.13.97:999 for domain xiaoxi.ilovefyy.top
Starting Server v0.5.2...
16:23:05 : Debug main.c:132	Chroot to /tmp
08:23:05 : Debug main.c:142	Change to user nobody
```



