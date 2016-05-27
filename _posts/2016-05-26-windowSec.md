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
# 常用命令操作
	
	net user username password /add 增加一个username的用户，密码为password
	net localgroup Administrators username /add 把username添加到Administrator组




----

