---
title: linux加固
time: 2016.05.28 12:47:00
layout: post
catalog: true
tags:
- linux加固
- Security
excerpt: 蓝盾比赛的一些准备与总结



---


1.用户设置：（删除具体用户）
用户名：口令：UID：GID：用户注释：用户主目录：命令解释程序（dev/bash；bin/sh）；
特别注意命令解释程序：dev/bash或bin/sh改为dev/null和bin/false
usermod -s /dev/null username        #更改
groupdel groupname        #删除用户组
userdel username            #删除用户，lp，sync，shutdown，halt，news，uncp，operator，games，gopher

useradd XX XX

2.密码设置：（弱口令）
awk  -F:'($3==0){print $1}' /etc/passwd        #查找UID为0的用户
awk  -F:'($2==0){print $1}' /etc/passwd        #查询密码为空用户
密码长度限制
修改root用户密码：
passwd

3.登陆设置
用户登录限制：
vi /etc/passwd
..../sbin/nologin        #作用：禁止该用户使用bash或shell系统，但任可使用系统资源    
touch /etc/nologin    #除root以外用户禁止登陆
禁止以下系统用户登录：
daemon，bin，sys，adm，lp，uucp，nuucp，smmsp等

远程登陆限制：
vi /etc/ssh/sshd_config
PermitRootLogin no        #禁止root远程登陆
Port 2222        #修改ssh默认端口
重启sshd服务：
service sshd restart
修改配置仅允许管理员console登陆：
vi /etc/securetty
CONSOLE=/dev/tty01        #添加

4.权限设置
用户权限：
chmod +wx filename        #给当前用户增加权限
具体语法：chmod [who] [+ | - | =] [mode] filename/foldername
关键目录权限：        ---flag文件夹
ls -al /etc/        #查看目录权限
chmod 644  /etc/passwd
chmod 600  /etc/shadow
chmod 644  /etc/group

5.服务设置：
不响应ICMP：
cat /proc/sys/net/ipv4/icm_echo_ignore_all
echo 1 > /proc/sys/net/ipv4/icmp_echo_ignore_all

无效服务关闭：
ftp，telnet，shell，login，exec，talk，ntalk，imap，pop-2，pop-3，finger，
auth，rpc，portmap

systemctl status shell	#可查看某服务是否开启
systemctl start shell	#可开启某服务
systemctl list-units -t service		#查看所有被激活的服务
systemctl is-enabled xxx.service	#查看服务是否被开启 
snmp服务加固


6.信息伪造：
伪造flag

7.信息隐藏：


8.系统检测：
异常进程：异常登录：异常服务：异常文件：系统日志

异常文件：
find / -name "..*" -print            #查找异常隐含文件
find / -nouser -o nogroup -print        #查找没有属主的文件
find / -type b -print | grep -v ' ^/dev/'        #查找非dev下的设备文件
find / -name .netrc        #查找netrc文件
find / -name .rhosts -print        #查找rhost文件

w	#可查看当前系统登录用户
who	#查看当前用户
last	#查看曾经登陆用户
unermod -L	#锁定用户账号


9.iptable设置：
端口配置；
关闭所有的 INPUT FORWARD OUTPUT 只对某些端口开放。

    iptables -P INPUT DROP
    iptables -P FORWARD DROP
    iptables -P OUTPUT DROP

只打开80端口

    iptables -A INPUT -p tcp --dport 80 -j ACCEPT
    iptables -A OUTPUT -p tcp --sport 80 -j ACCEPT

ping包管理；
允许自己ping别人，别人ping不通自己
在出去的端口上：iptables -A OUTPUT -p icmp --icmp-type 8 -j ACCEPT
在进来的端口上：iptables -A INPUT -p icmp --icmp-type 0 -j ACCEPT
对于127.0.0.1比较特殊，我们需要明确定义它
			iptables -A INPUT -s 127.0.0.1 -d 127.0.0.1 -j ACCEPT
			iptables -A OUTPUT -s 127.0.0.1 -d 127.0.0.1 -j ACCEPT

限制IP访问；
禁止192.168.1.2访问我（如果发现有人连接到我的电脑，禁止他）

 iptables -A INPUT -p tcp -s 192.168.1.2 -j DROP

防止无效的数据包
 iptables -A OUTPUT -p tcp –sport 22 -m state –state ESTABLISHED -j ACCEPT

----------------------------------------------

一些补充


主机探测扫描
arp_sweep 使用arp请求枚举活跃主机（只能探测同一子网中的活跃主机）
udp_sweep发送udp探查主机是否活跃
show auxiliary  显示工具路径
nmap -sh(选项) IP


破解windows的攻击
load mimikatz
msv
kerberos
shell
echo hello.dajiahao>hello.txt
type hello.txt
net user abc 123456 /add
net localgroup administrator abc /add
net localgroup “remote desktop users" abc /add


开启msf
service postgresql start
msfconsole


