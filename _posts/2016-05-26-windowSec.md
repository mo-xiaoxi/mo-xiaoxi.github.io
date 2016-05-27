---
title: Windows渗透工具学习笔记
time: 2016.05.26 12:47:00
layout: post
tags:
- windows
- 工具
- 安全相关
excerpt:  windows相关的渗透工具与一些常见的攻击操作

---

# GET HASH
Windows的系统密码hash默认情况下一般由两部分组成：第一部分是LM-hash，第二部分是NTLM-hash。
## 工具使用
1. 2000下可以用pwdump7.exe来抓取系统用户的hash，命令格式：pwdump7.exe >2000hash.txt，意思为抓取所有用户hash，并写入2000hash.txt这个文本文件
2. 也可以用SAMInside自带的小工具GetHashes.exe，命令格式：GetHashes.exe $local >2000.txt，意思是抓取所有用户hash并写入2000.txt这个文本文件
3. 还可以用图形界面的SAMInside，打开SAMInside，点击“File”，然后点击“Import local Users via Scheduler”，稍等一会就成功抓取到hash了。	因为这个是利用Windows的计划任务来抓取的，所以Task Scheduler服务必须启动，否则抓不出来。抓到hash之后还要导出，以方便用其它更强大的破解工具进行破解工作。如果要导出所有用户的hash，就点击“File”，然后点击“Export Users to PWDUMP File…”，然后保存为txt文本即可。如果只需要其中一个用户的hash，就选“Export Selected Users to PWDUMP File…”，同样保存为txt文本即可。
4. Windows 2000下的另一个得到管理员密码的方法，用aio.exe（aio是All In One的缩写，是一些小工具的集合）直接读取内存中的密码，Windows 2003 SP1、SP2补丁没有打的话，也可以这样读取出来密码明文。命令格式：aio.exe -findpassword，
5. LC5：就依次打开“Session”->“Import”->“Local machine”稍等片刻就可以成功抓取到hash了，如果你要导入破解hash，就选“Import from file”->“From PWDUMP file”导入即可进行破解
6. Cain：点选“Cracker”->“LM&NTLM Hashes”，然后点下右边空白处，蓝色+号按钮即可激活，然后点击它，弹出“Add NT Hashes from”->“Import Hashes from local system”->勾选“Include Password History Hashes”，然后Next，hash就抓出来了
7. ppa：要用ppa配合彩虹表破解的方法也比较简单，Attack选择“Rainbow”->“NTLM attack”->“Rainbow tables list…”->“Add”选择导入彩虹表文件
8. Ophcrack

-----
# 常用命令操作：
### 最常用
	net user username password /add 增加一个username的用户，密码为password
	net localgroup Administrators username /add 把username添加到Administrator组
	net start telnet 开对方的TELNET服务
	net use Z:\127.0.0.1c$ 映射对方的C盘
	net user guest /active:yes 将Guest用户激活
	net user guest password 把guest的密码改为password
	net user username /delete  删除用户
	net password 密码 更改系统登陆密码 
	netstat -a 查看开启了哪些端口,常用netstat -an 
	netstat -n 查看端口的网络连接情况，常用netstat -an 
	netstat -v 查看正在进行的工作
	如果在linux下，首先cat /etc/shadow
1.Net常用命令:

	(1)net share - 查看共享命令
   		net share ipc$ - 设置ipc$共享
   		net share ipc$ /del - 删除ipc$共享 (xp系统无法删除)
   		net share c$=c: - 设置c盘为共享
	(2)net user - 查看本地的用户列表
   		net user 用户名 密码 /add - 增加一个用户
   		net user 用户名 /add 或 net user 用户名 "" /add - 增加一个密码为空的用户
   		net user 用户名 /del - 删除某个用户名
   		net user 用户名 /active:yes(no) - 设置某个用户的状态为启用(禁用)
	(3)net localgroup administrators - 查看管理员组里的用户(即权限为管理员的用户)
   		net localgroup administrators 用户名 /add - 把某个用户增加到管理员组里
   		net localgroup administrators 用户名 /del - 从管理员组里删除某个用户
   		注意:1.增加到某个组里的用户必须是已经被创建过的用户.
        	2.增加到的组必须为存在的组.
	(4)net start - 查看已经启动的服务列表
   		net start 服务名 - 开启某个服务   注意:要想成功的开启一个服务,前提是它被停用了,而不是被禁止
   		net stop 服务名  - 停止某个服务   注意:停止的服务必须是已经启动的,而不是已经停止或是被禁止的
	(5)net use \\ip地址\ipc$ "密码" /user:用户名 - 和某个ip地址建立一个ipc$连接(ipc$入侵)
   		net use \\ip地址\ipc$ /del - 删除建立的ipc$连接
   		命令成功与否的前提:1.对方操作系统是否为NT以上的(除xp外)
                      2.对方系统是否开启了ipc$共享
                      3.输入的用户名和密码是否正确
	(6)net use h: \\ip地址\c$ - 将对方c盘映射到本地的h盘
   		net use h: /del -删除映射到本地的磁盘
   		注意:1.要映射到本地的磁盘名不能与本地现有的磁盘名重复(冲突)
        	2.想要映射对方的某个磁盘或目录的前提是对方的此磁盘或目录设置了共享





----

