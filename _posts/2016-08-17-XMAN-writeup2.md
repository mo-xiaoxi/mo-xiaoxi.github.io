---


title: XMAN writeup 2

time: 2016.08.17 15:22:00

layout: post

tags:

- Security
- XMAN
- CTF

excerpt: XMAN夏令营writeup2:这里是我一直想入门的二进制相关主题

---

# XMAN writeup 2

### 课上练习－CM1

[题目附件](http://xman.xctf.org.cn/media/task/cdaf8d32-a95c-4622-b4fb-f599c56ea9e4.exe)

下载下来，拉进IDA，然后F5，可以看到源码

![image](http://momomoxiaoxi.com/img/post/XMAN/2.png)

分析下源码，就是一个字符串验证，然后查看下Str2的值，为Thi5_i5_T0o_E4sy
输入下就得到了下面的答案

![image](http://momomoxiaoxi.com/img/post/XMAN/3.png)


### 