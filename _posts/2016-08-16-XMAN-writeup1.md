---
title: XMAN writeup 1 

time: 2016.08.15 15:22:00

layout: post
catalog: true
tags:
- Security
- CTF

excerpt: XMAN夏令营writeup1-MISC和Crypt相关，以及各类大杂项。


---

### 0x00类似培根

#### 题目描述

```
fIND PasswoRd:
fIrSt tHiNk OF thE PErSon wHo liVEs In diSguIse,
Who deals in SEcretS and tellS naUght But lIEs.
NExt, Tell Me whAT'S Always The LaSt tHiNg TO meND,ThE midDle oF MiDdle And End Of the end?
anD FinalLy give me The SounD oftEN hEArd DuriNg thE SEArch foR a hArD-to-FiNd WOrk.
NOW sTrinG theM ToGethEr, aNd aNswer me thiS,
Which Creature WouLd yoU be uNWiLLinG to kIss?
```

#### 思路：


```
#!usr/bin/env python
#!-*- coding:utf-8 -*-
def baconlike(bacon):
    l=len(bacon)
    out=''
    for i in range(0,l):
        if ord(bacon[i]) in range(ord('a'),ord('z')+1):
            out+='a'
        elif(ord(bacon[i]) in range(ord('A'),ord('Z')+1)):
            out+='b'     
    return out

if __name__ =='__main__':
    t=baconlike('fIND PasswoRd:fIrSt tHiNk OF thE PErSon wHo liVEs In diSguIse,Who deals in SEcretS and tellS naUght But lIEs.NExt, Tell Me whAT\'S Always The LaSt tHiNg TO meND,ThE midDle oF MiDdle And End Of the end?anD FinalLy give me The SounD oftEN hEArd DuriNg thE SEArch foR a hArD-to-FiNd WOrk.NOW sTrinG theM ToGethEr, aNd aNswer me thiS,Which Creature WouLd yoU be uNWiLLinG to kIss?')
    print t
```

先换成ab，

abbbbaaaaabaababaabababbaabbbabaaabaaabbabaaabaabaabaaaaaaaaabbaaaabaaaaaaabaabaaabaaabbabbaabaaabaaabbbbaaaaabaababaabababbaabbbabaaabaaabbabaaabaabaabaaaaaaaaabbaaaabaaaaaaabaabaaabaaabbabbaabaaabaaabbbbaaaaabaababaabababbaabbbabaaabaaabbabaaabaabaabaaaaaaaaabbaaaabaaaaaaabaabaaabaaabbabbaabaaabaa

然后培根解密下
PASSWORD IS I AM EASENSE
PASSWORDISIAMEASENSE 
PASSWORD ISIAMEASENSE

#### flag

iameasense

-----

### 0x01 诱人的wifi

#### 题目描述：

以下是来自某寝室的一段对话：

A:“B，你开个wifi吧！”

B:“这才10号，流量就用光了？”

C:“是啊，那些用图片刷群的伤不起啊！”

B:“A，你怎么不开？”

A:“这不是懒得下去了么……C，你开一个呗～”

C：“我系统是win20的，开不了……D？”

D：“zzzzzzzZZZZZZZZ……”

没办法，小A打算蹭隔壁的wifi了，可是wifi密码是多少呢？小A还听说，隔壁的wifi密码就是这题的flag呢！

（提示：wifi密码为8位数字，第一位为1）

#### 思路：

WPA2握手包破解，可以使用EWSA、hashcat或者其他握手包破解工具破解wifi密码。
导入EWSA破解即可。

#### flag:

11223344

------
### 0x02 playfair密码

#### 题目描述：

Ciphertext: RSXIXTRVFXSX

Key：THEQUICKBROWNFOXJUMPSOVERALAZYDOG

Plaintext:？

#### 思路：

丢进cap4解密即可

![image](http://momomoxiaoxi.com/img/post/XMAN/1.png)

#### flag:

kaoroubanfan

-----

### 0x03 EASY_RSA

下载下来有两个文件，一个公钥，一个加密后的问题。

首先通过openssl查看公钥信息

```
root@kali:~/桌面/XMAN/EASY_RSA# openssl rsa -noout -text -inform PEM -in public.pem -pubin -modulus
Public-Key: (256 bit)
Modulus:
    00:c2:63:6a:e5:c3:d8:e4:3f:fb:97:ab:09:02:8f:
    1a:ac:6c:0b:f6:cd:3d:70:eb:ca:28:1b:ff:e9:7f:
    be:30:dd
Exponent: 65537 (0x10001)
Modulus=C2636AE5C3D8E43FFB97AB09028F1AAC6C0BF6CD3D70EBCA281BFFE97FBE30DD
```

得到e为65537 n=C2636AE5C3D8E43FFB97AB09028F1AAC6C0BF6CD3D70EBCA281BFFE97FBE30DD
将n转为10进制：

```
➜  ~ python
Python 2.7.11 (default, Dec 16 2015, 11:23:08) 
[GCC 4.2.1 Compatible Apple LLVM 7.0.0 (clang-700.1.76)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> print int('C2636AE5C3D8E43FFB97AB09028F1AAC6C0BF6CD3D70EBCA281BFFE97FBE30DD',16)
87924348264132406875276140514499937145050893665602592992418171647042491658461
>>> 
```
然后到[FactorDB](http://www.factordb.com/index.php?query=87924348264132406875276140514499937145050893665602592992418171647042491658461)解一下，得到p=319576316814478949870590164193048041239 q= 275127860351348928173285174381581152299

接下来使用[rsatool](https://github.com/ius/rsatool)生成private.key

```
root@kali:~/桌面/XMAN/EASY_RSA/rsatool-master# ./rsatool.py -p 319576316814478949870590164193048041239 -q 275127860351348928173285174381581152299 -o private.key
Using (p, q) to initialise RSA instance
n =
c2636ae5c3d8e43ffb97ab09028f1aac6c0bf6cd3d70ebca281bffe97fbe30dd
e = 65537 (0x10001)
d =
1806799bd44ce649122b78b43060c786f8b77fb1593e0842da063ba0d8728bf1
p = 319576316814478949870590164193048041239 (0xf06c28e91c8922b9c236e23560c09717)
q = 275127860351348928173285174381581152299 (0xcefbb2cf7e18a98ebedc36e3e7c3b02b)
Saving PEM as private.key
```

然后使用openssl解密：

```
root@kali:~/桌面/XMAN/EASY_RSA/rsatool-master# cat private.key 
-----BEGIN RSA PRIVATE KEY-----
MIGrAgEAAiEAwmNq5cPY5D/7l6sJAo8arGwL9s09cOvKKBv/6X++MN0CAwEAAQIgGAZ5m9RM5kkS
K3i0MGDHhvi3f7FZPghC2gY7oNhyi/ECEQDwbCjpHIkiucI24jVgwJcXAhEAzvuyz34YqY6+3Dbj
58OwKwIQaN4kshl6T6VK63mb4snenQIRAJulRkclqWIHx5pNZIAp9VUCEQCzufLwcMVFMaQ7MGqz
qBUl
-----END RSA PRIVATE KEY-----
root@kali:~/桌面/XMAN/EASY_RSA/rsatool-master# cd ..
root@kali:~/桌面/XMAN/EASY_RSA# ls
1.py  flag.enc  public.pem  rsatool-master
root@kali:~/桌面/XMAN/EASY_RSA# mv ./rsatool-master/private.key  .
root@kali:~/桌面/XMAN/EASY_RSA# ls
1.py  flag.enc  private.key  public.pem  rsatool-master
root@kali:~/桌面/XMAN/EASY_RSA# cat flag.enc 
���r�&{Ϙ���a��ōcZ�2�ݫ���Kroot@kali:~/桌面/XMAN/EASY_RSA# ls
1.py  flag.enc  private.key  public.pem  rsatool-master
root@kali:~/桌面/XMAN/EASY_RSA# openssl rsautl -decrypt -in flag.enc -inkey private.key  -out flag.txt
root@kali:~/桌面/XMAN/EASY_RSA# cat flag.txt 
flag{ea5y_r5a_is_0k}root@kali:~/桌面/XMAN/EASY_RSA#   
```

-------

### 0x04 RSA小公钥指数攻击

#### 题目描述：
RSA小公钥指数攻击

#### 思路：

首先查看公钥：

```
root@kali:~/桌面/XMAN/RSAxiao# openssl rsa -noout -text -inform PEM -in pubkey.pem -pubin -modulus
Public-Key: (4096 bit)
Modulus:
    00:b0:be:e5:e3:e9:e5:a7:e8:d0:0b:49:33:55:c6:
    18:fc:8c:7d:7d:03:b8:2e:40:99:51:c1:82:f3:98:
    de:e3:10:45:80:e7:ba:70:d3:83:ae:53:11:47:56:
    56:e8:a9:64:d3:80:cb:15:7f:48:c9:51:ad:fa:65:
    db:0b:12:2c:a4:0e:42:fa:70:91:89:b7:19:a4:f0:
    d7:46:e2:f6:06:9b:af:11:ce:bd:65:0f:14:b9:3c:
    97:73:52:fd:13:b1:ee:a6:d6:e1:da:77:55:02:ab:
    ff:89:d3:a8:b3:61:5f:d0:db:49:b8:8a:97:6b:c2:
    05:68:48:92:84:e1:81:f6:f1:1e:27:08:91:c8:ef:
    80:01:7b:ad:23:8e:36:30:39:a4:58:47:0f:17:49:
    10:1b:c2:99:49:d3:a4:f4:03:8d:46:39:38:85:15:
    79:c7:52:5a:69:98:4f:15:b5:66:7f:34:20:9b:70:
    eb:26:11:36:94:7f:a1:23:e5:49:df:ff:00:60:18:
    83:af:d9:36:fe:41:1e:00:6e:4e:93:d1:a0:0b:0f:
    ea:54:1b:bf:c8:c5:18:6c:b6:22:05:03:a9:4b:24:
    13:11:0d:64:0c:77:ea:54:ba:32:20:fc:8f:4c:c6:
    ce:77:15:1e:29:b3:e0:65:78:c4:78:bd:1b:eb:e0:
    45:89:ef:9a:19:7f:6f:80:6d:b8:b3:ec:d8:26:ca:
    d2:4f:53:24:cc:de:c6:e8:fe:ad:2c:21:50:06:86:
    02:c8:dc:dc:59:40:2c:ca:c9:42:4b:79:00:48:cc:
    dd:93:27:06:80:95:ef:a0:10:b7:f1:96:c7:4b:a8:
    c3:7b:12:8f:9e:14:11:75:16:33:f7:8b:7b:9e:56:
    f7:1f:77:a1:b4:da:ad:3f:c5:4b:5e:7e:f9:35:d9:
    a7:2f:b1:76:75:97:65:52:2b:4b:bc:02:e3:14:d5:
    c0:6b:64:d5:05:4b:7b:09:6c:60:12:36:e6:cc:f4:
    5b:5e:61:1c:80:5d:33:5d:ba:b0:c3:5d:22:6c:c2:
    08:d8:ce:47:36:ba:39:a0:35:44:26:fa:e0:06:c7:
    fe:52:d5:26:7d:cf:b9:c3:88:4f:51:fd:df:df:4a:
    97:94:bc:fe:0e:15:57:11:37:49:e6:c8:ef:42:1d:
    ba:26:3a:ff:68:73:9c:e0:0e:d8:0f:d0:02:2e:f9:
    2d:34:88:f7:6d:eb:62:bd:ef:7b:ea:60:26:f2:2a:
    1d:25:aa:2a:92:d1:24:41:4a:80:21:fe:0c:17:4b:
    98:03:e6:bb:5f:ad:75:e1:86:a9:46:a1:72:80:77:
    0f:12:43:f4:38:74:46:cc:ce:b2:22:2a:96:5c:c3:
    0b:39:29
Exponent: 3 (0x3)
Modulus=B0BEE5E3E9E5A7E8D00B493355C618FC8C7D7D03B82E409951C182F398DEE3104580E7BA70D383AE5311475656E8A964D380CB157F48C951ADFA65DB0B122CA40E42FA709189B719A4F0D746E2F6069BAF11CEBD650F14B93C977352FD13B1EEA6D6E1DA775502ABFF89D3A8B3615FD0DB49B88A976BC20568489284E181F6F11E270891C8EF80017BAD238E363039A458470F1749101BC29949D3A4F4038D463938851579C7525A69984F15B5667F34209B70EB261136947FA123E549DFFF00601883AFD936FE411E006E4E93D1A00B0FEA541BBFC8C5186CB6220503A94B2413110D640C77EA54BA3220FC8F4CC6CE77151E29B3E06578C478BD1BEBE04589EF9A197F6F806DB8B3ECD826CAD24F5324CCDEC6E8FEAD2C2150068602C8DCDC59402CCAC9424B790048CCDD9327068095EFA010B7F196C74BA8C37B128F9E1411751633F78B7B9E56F71F77A1B4DAAD3FC54B5E7EF935D9A72FB176759765522B4BBC02E314D5C06B64D5054B7B096C601236E6CCF45B5E611C805D335DBAB0C35D226CC208D8CE4736BA39A0354426FAE006C7FE52D5267DCFB9C3884F51FDDFDF4A9794BCFE0E1557113749E6C8EF421DBA263AFF68739CE00ED80FD0022EF92D3488F76DEB62BDEF7BEA6026F22A1D25AA2A92D124414A8021FE0C174B9803E6BB5FAD75E186A946A17280770F1243F4387446CCCEB2222A965CC30B3929
```
可以看到e很小

#### flag:



-------

### 0x05 base64

#### 题目描述
```
QmFzZTY0IGlzIGEgZ3JvdXAgb2Ygc2ltaWxhciBiaW5hcnktdG8tdGV4dCBlbmNvZGluZyBzY2hlbWVzIHRoYXQgcmVwcmVzZW50IGJpbmFyeSBkYXRhIGluIGFuIEFTQ0lJIHN0cmluZyBmb3JtYXQgYnkgdHJhbnNsYXRpbmcgaXQgaW50byBhIHJhZGl4LTY0IHJlcHJlc2VudGF0aW9uLiBUaGUgdGVybSBCYXNlNjQgb3JpZ2luYXRlcyBmcm9tIGEgc3BlY2lmaWMgTUlNRSBjb250ZW50IHRyYW5zZmVyIGVuY29kaW5nLgpUaGUgcGFydGljdWxhciBzZXQgb2YgNjQgY2hhcmFjdGVycyBjaG9zZW4gdG8gcmVwcmVzZW50IHRoZSA2NCBwbGFjZS12YWx1ZXMgZm9yIHRoZSBiYXNlIHZhcmllcyBiZXR3ZWVuIGltcGxlbWVudGF0aW9ucy4gVGhlIGdlbmVyYWwgc3RyYXRlZ3kgaXMgdG8gY2hvb3NlIDY0IGNoYXJhY3RlcnMgdGhhdCBhcmUgYm90aCBtZW1iZXJzIG9mIGEgc3Vic2V0IGNvbW1vbiB0byBtb3N0IGVuY29kaW5ncw==
IGFuZCBhbHNvIHByaW50YWJsZS4gVGhpcyBjb21iaW5hdGlvbiBsZWF2ZXMgdGhlIGRhdGEgdW5saWtlbHkgdG8gYmUgbW9kaWZpZWQgaW4gdHJhbnNpdCB0aHJvdWdoIGluZm9ybWF0aW9uIHN5c3RlbXM=
IHN1Y2ggYXMgZW1haWw=
IHRoYXQgd2VyZSB0cmFkaXRpb25hbGx5IG5vdCA4LWJpdCBjbGVhbi5bMV0gRm9yIGV4YW1wbGU=
IGGoQ3o=
IGFuZCAwqEM5IGZvciB0aGUgZmlyc3QgNjIgdmFsdWVzLiBPdGhlciB2YXJpYXRpb25zIHNoYXJlIHRoaXMgcHJvcGVydHkgYnV0IGRpZmZlciBpbiB0aGUgc3ltYm9scyBjaG9zZW4gZm9yIHRoZSBsYXN0IHR3byB2YWx1ZXM7IGFuIGV4YW1wbGUgaXMgVVRGLTcuCgpUaGUgZWFybGllc3QgaW5zdGFuY2VzIG9mIHRoaXMgdHlwZSBvZiBlbmNvZGluZyB3ZXJlIGNyZWF0ZWQgZm9yIGRpYWx1cCBjb21tdW5pY2F0aW9uIGJldHdlZW4gc3lzdGVtcyBydW5uaW5nIHRoZSBzYW1lIE9TIKGqIGUuZy4=
IEJpbkhleCBmb3IgdGhlIFRSUy04MCAobGF0ZXIgYWRhcHRlZCBmb3IgdGhlIE1hY2ludG9zaCkgoaogYW5kIGNvdWxkIHRoZXJlZm9yZSBtYWtlIG1vcmUgYXNzdW1wdGlvbnMgYWJvdXQgd2hhdCBjaGFyYWN0ZXJzIHdlcmUgc2FmZSB0byB1c2UuIEZvciBpbnN0YW5jZQ==
IHV1ZW5jb2RlIHVzZXMgdXBwZXJjYXNlIGxldHRlcnM=
IGRpZ2l0cw==
IGFuZCBtYW55IHB1bmN0dWF0aW9uIGNoYXJhY3RlcnM=
IHByb3Bvc2VkIGJ5IFJGQyA5ODkgaW4gMTk4Ny4gUEVNIGRlZmluZXMgYSAicHJpbnRhYmxlIGVuY29kaW5nIiBzY2hlbWUgdGhhdCB1c2VzIEJhc2U2NCBlbmNvZGluZyB0byB0cmFuc2Zvcm0gYW4gYXJiaXRyYXJ5IHNlcXVlbmNlIG9mIG9jdGV0cyB0byBhIGZvcm1hdCB0aGF0IGNhbiBiZSBleHByZXNzZWQgaW4gc2hvcnQgbGluZXMgb2YgNi1iaXQgY2hhcmFjdGVycw==
IGFzIHJlcXVpcmVkIGJ5IHRyYW5zZmVyIHByb3RvY29scyBzdWNoIGFzIFNNVFAuWzZdCgpUaGUgY3VycmVudCB2ZXJzaW9uIG9mIFBFTSAoc3BlY2lmaWVkIGluIFJGQyAxNDIxKSB1c2VzIGEgNjQtY2hhcmFjdGVyIGFscGhhYmV0IGNvbnNpc3Rpbmcgb2YgdXBwZXItIGFuZCBsb3dlci1jYXNlIFJvbWFuIGxldHRlcnMgKEGoQ1o=
IHRoZSBudW1lcmFscyAoMKhDOSk=
IGFuZCB0aGUgIisiIGFuZCAiLyIgc3ltYm9scy4gVGhlICI9IiBzeW1ib2wgaXMgYWxzbyB1c2VkIGFzIGEgc3BlY2lhbCBzdWZmaXggY29kZS5bMl0gVGhlIG9yaWdpbmFsIHNwZWNpZmljYXRpb24=
IFJGQyA5ODk=
IGFkZGl0aW9uYWxseSB1c2VkIHRoZSAiKiIgc3ltYm9sIHRvIGRlbGltaXQgZW5jb2RlZCBidXQgdW5lbmNyeXB0ZWQgZGF0YSB3aXRoaW4gdGhlIG91dHB1dCBzdHJlYW0uCgpUbyBjb252ZXJ0IGRhdGEgdG8gUEVNIHByaW50YWJsZSBlbmNvZGluZx==
IHRoZSBmaXJzdCBieXRlIGlzIHBsYWNlZCBpbiB0aGUgbW9zdCBzaWduaWZpY2FudCBlaWdodCBiaXRzIG9mIGEgMjQtYml0IGJ1ZmZlcp==
IHRoZSBuZXh0IGluIHRoZSBtaWRkbGUgZWlnaHS=
IGFuZCB0aGUgdGhpcmQgaW4gdGhlIGxlYXN0IHNpZ25pZmljYW50IGVpZ2h0IGJpdHMuIElmIHRoZXJlIGFyZSBmZXdlciB0aGFuIHRocmVlIGJ5dGVzIGxlZnQgdG8gZW5jb2RlIChvciBpbiB0b3RhbCl=
IHRoZSByZW1haW5pbmcgYnVmZmVyIGJpdHMgd2lsbCBiZSB6ZXJvLiBUaGUgYnVmZmVyIGlzIHRoZW4gdXNlZL==
IHNpeCBiaXRzIGF0IGEgdGltZR==
IG1vc3Qgc2lnbmlmaWNhbnQgZmlyc3S=
IGFzIGluZGljZXMgaW50byB0aGUgc3RyaW5nOiAiQUJDREVGR0hJSktMTU5PUFFSU1RVVldYWVphYmNkZWZnaGlqa2xtbm9wcXJzdHV2d3h5ejAxMjM0NTY3ODkrLyI=
IGFuZCB0aGUgaW5kaWNhdGVkIGNoYXJhY3RlciBpcyBvdXRwdXQuCgpUaGUgcHJvY2VzcyBpcyByZXBlYXRlZCBvbiB0aGUgcmVtYWluaW5nIGRhdGEgdW50aWwgZmV3ZXIgdGhhbiBmb3VyIG9jdGV0cyByZW1haW4uIElmIHRocmVlIG9jdGV0cyByZW1haW5=
IHRoZSBpbnB1dCBkYXRhIGlzIHJpZ2h0LXBhZGRlZCB3aXRoIHplcm8gYml0cyB0byBmb3JtIGFuIGludGVncmFsIG11bHRpcGxlIG9mIHNpeCBiaXRzLgoKQWZ0ZXIgZW5jb2RpbmcgdGhlIG5vbi1wYWRkZWQgZGF0YW==
IGlmIHR3byBvY3RldHMgb2YgdGhlIDI0LWJpdCBidWZmZXIgYXJlIHBhZGRlZC16ZXJvc3==
IHR3byAiPSIgY2hhcmFjdGVycyBhcmUgYXBwZW5kZWQgdG8gdGhlIG91dHB1dDsgaWYgb25lIG9jdGV0IG9mIHRoZSAyNC1iaXQgYnVmZmVyIGlzIGZpbGxlZCB3aXRoIHBhZGRlZC16ZXJvc3==
IG9uZSAiPSIgY2hhcmFjdGVyIGlzIGFwcGVuZGVkLiBUaGlzIHNpZ25hbHMgdGhlIGRlY29kZXIgdGhhdCB0aGUgemVybyBiaXRzIGFkZGVkIGR1ZSB0byBwYWRkaW5nIHNob3VsZCBiZSBleGNsdWRlZCBmcm9tIHRoZSByZWNvbnN0cnVjdGVkIGRhdGEuIFRoaXMgYWxzbyBndWFyYW50ZWVzIHRoYXQgdGhlIGVuY29kZWQgb3V0cHV0IGxlbmd0aCBpcyBhIG11bHRpcGxlIG9mIDQgYnl0ZXMuCgpQRU0gcmVxdWlyZXMgdGhhdCBhbGwgZW5jb2RlZCBsaW5lcyBjb25zaXN0IG9mIGV4YWN0bHkgNjQgcHJpbnRhYmxlIGNoYXJhY3RlcnO=
IHdoaWNoIG1heSBjb250YWluIGZld2VyIHByaW50YWJsZSBjaGFyYWN0ZXJzLiBMaW5lcyBhcmUgZGVsaW1pdGVkIGJ5IHdoaXRlc3BhY2UgY2hhcmFjdGVycyBhY2NvcmRpbmcgdG8gbG9jYWwgKHBsYXRmb3JtLXNwZWNpZmljKSBjb252ZW50aW9ucy4KCk1JTUVbZWRpdF0KTWFpbiBhcnRpY2xlOiBNSU1FClRoZSBNSU1FIChNdWx0aXB1cnBvc2UgSW50ZXJuZXQgTWFpbCBFeHRlbnNpb25zKSBzcGVjaWZpY2F0aW9uIGxpc3RzIEJhc2U2NCBhcyBvbmUgb2YgdHdvIGJpbmFyeS10by10ZXh0IGVuY29kaW5nIHNjaGVtZXMgKHRoZSBvdGhlciBiZWluZyBxdW90ZWQtcHJpbnRhYmxlKS5bM10gTUlNRSdzIEJhc2U2NCBlbmNvZGluZyBpcyBiYXNlZCBvbiB0aGF0IG9mIHRoZSBSRkMgMTQyMSB2ZXJzaW9uIG9mIFBFTTogaXQgdXNlcyB0aGUgc2FtZSA2NC1jaGFyYWN0ZXIgYWxwaGFiZXQgYW5kIGVuY29kaW5nIG1lY2hhbmlzbSBhcyBQRU3=
IGFuZCB1c2VzIHRoZSAiPSIgc3ltYm9sIGZvciBvdXRwdXQgcGFkZGluZyBpbiB0aGUgc2FtZSB3YXl=
IGFzIGRlc2NyaWJlZCBhdCBSRkMgMjA0NS4KCk1JTUUgZG9lcyBub3Qgc3BlY2lmeSBhIGZpeGVkIGxlbmd0aCBmb3IgQmFzZTY0LWVuY29kZWQgbGluZXM=
IHRoZSBhY3R1YWwgbGVuZ3RoIG9mIE1JTUUtY29tcGxpYW50IEJhc2U2NC1lbmNvZGVkIGJpbmFyeSBkYXRhIGlzIHVzdWFsbHkgYWJvdXQgMTM3JSBvZiB0aGUgb3JpZ2luYWwgZGF0YSBsZW5ndGg=
IHRob3VnaCBmb3IgdmVyeSBzaG9ydCBtZXNzYWdlcyB0aGUgb3ZlcmhlYWQgY2FuIGJlIG11Y2ggaGlnaGVyIGR1ZSB0byB0aGUgb3ZlcmhlYWQgb2YgdGhlIGhlYWRlcnMuIFZlcnkgcm91Z2hseZ==
IHRoZSBmaW5hbCBzaXplIG9mIEJhc2U2NC1lbmNvZGVkIGJpbmFyeSBkYXRhIGlzIGVxdWFsIHRvIDEuMzcgdGltZXMgdGhlIG9yaWdpbmFsIGRhdGEgc2l6ZSArIDgxNCBieXRlcyAoZm9yIGhlYWRlcnMpLiBUaGUgc2l6ZSBvZiB0aGUgZGVjb2RlZCBkYXRhIGNhbiBiZSBhcHByb3hpbWF0ZWQgd2l0aCB0aGlzIGZvcm11bGE6CgpieXRlcyA9IChzdHJpbmdfbGVuZ3RoKGVuY29kZWRfc3RyaW5nKSAtIDgxNCkgLyAxLjM3ClVURi03W2VkaXRdCk1haW4gYXJ0aWNsZTogVVRGLTcKVVRGLTc=
IGRlc2NyaWJlZCBmaXJzdCBpbiBSRkMgMTY0Mh==
IGFuZCB0aGUgIj0iIGNoYXJhY3RlciBpcyByZXNlcnZlZCBpbiB0aGF0IGNvbnRleHQgYXMgdGhlIGVzY2FwZSBjaGFyYWN0ZXIgZm9yICJxdW90ZWQtcHJpbnRhYmxlIiBlbmNvZGluZy4gTW9kaWZpZWQgQmFzZTY0IHNpbXBseSBvbWl0cyB0aGUgcGFkZGluZyBhbmQgZW5kcyBpbW1lZGlhdGVseSBhZnRlciB0aGUgbGFzdCBCYXNlNjQgZGlnaXQgY29udGFpbmluZyB1c2VmdWwgYml0cyBsZWF2aW5nIHVwIHRvIHRocmVlIHVudXNlZCBiaXRzIGluIHRoZSBsYXN0IEJhc2U2NCBkaWdpdC4KCk9wZW5QR1BbZWRpdF0KTWFpbiBhcnRpY2xlOiBPcGVuUEdQCk9wZW5QR1B=
IGRlc2NyaWJlZCBpbiBSRkMgNDg4ME==
IGRlc2NyaWJlcyBSYWRpeC02NCBlbmNvZGluZ9==
IHVzaW5nIGFuIGFkZGl0aW9uYWwgIj0iIHN5bWJvbCBhcyBzZXBhcmF0b3I=
IGFwcGVuZGVkIHRvIHRoZSBlbmNvZGVkIG91dHB1dCBkYXRhLls5XQoKUkZDIDM1NDhbZWRpdF0KUkZDIDM1NDh=
IGVudGl0bGVkIFRoZSBCYXNlMTZ=
IEJhc2UzMj==
IGFuZCBCYXNlNjQgRGF0YSBFbmNvZGluZ3N=
IGlzIGFuIGluZm9ybWF0aW9uYWwgKG5vbi1ub3JtYXRpdmUpIG1lbW8gdGhhdCBhdHRlbXB0cyB0byB1bmlmeSB0aGUgUkZDIDE0MjEgYW5kIFJGQyAyMDQ1IHNwZWNpZmljYXRpb25zIG9mIEJhc2U2NCBlbmNvZGluZ3O=
IGFsdGVybmF0aXZlLWFscGhhYmV0IGVuY29kaW5ncz==
IGFuZCB0aGUgc2VsZG9tLXVzZWQgQmFzZTMyIGFuZCBCYXNlMTYgZW5jb2RpbmdzLgoKVW5sZXNzIGltcGxlbWVudGF0aW9ucyBhcmUgd3JpdHRlbiB0byBhIHNwZWNpZmljYXRpb24gdGhhdCByZWZlcnMgdG8gUkZDIDM1NDggYW5kIHNwZWNpZmljYWxseSByZXF1aXJlcyBvdGhlcndpc2V=
IGFuZCBpdCBhbHNvIGRlY2xhcmVzIHRoYXQgZGVjb2RlciBpbXBsZW1lbnRhdGlvbnMgbXVzdCByZWplY3QgZGF0YSB0aGF0IGNvbnRhaW4gY2hhcmFjdGVycyBvdXRzaWRlIHRoZSBlbmNvZGluZyBhbHBoYWJldC5bNF0KClJGQyA0NjQ4W2VkaXRdClRoaXMgUkZDIG9ic29sZXRlcyBSRkMgMzU0OCBhbmQgZm9jdXNlcyBvbiBCYXNlNjQvMzIvMTY6CgpUaGlzIGRvY3VtZW50IGRlc2NyaWJlcyB0aGUgY29tbW9ubHkgdXNlZCBCYXNlNjQ=
IEJhc2UzMl==
IGFuZCBCYXNlMTYgZW5jb2Rpbmcgc2NoZW1lcy4gSXQgYWxzbyBkaXNjdXNzZXMgdGhlIHVzZSBvZiBsaW5lLWZlZWRzIGluIGVuY29kZWQgZGF0Yf==
IHVzZSBvZiBwYWRkaW5nIGluIGVuY29kZWQgZGF0YW==
IHVzZSBvZiBub24tYWxwaGFiZXQgY2hhcmFjdGVycyBpbiBlbmNvZGVkIGRhdGG=
IHNpbmNlIHRoZW4gdGhlIGZpbGVuYW1lcyBjb3VsZCBiZSB1c2VkIGluIFVSTHMgYWxzby4KClVSTCBhcHBsaWNhdGlvbnNbZWRpdF0KQmFzZTY0IGVuY29kaW5nIGNhbiBiZSBoZWxwZnVsIHdoZW4gZmFpcmx5IGxlbmd0aHkgaWRlbnRpZnlpbmcgaW5mb3JtYXRpb24gaXMgdXNlZCBpbiBhbiBIVFRQIGVudmlyb25tZW50LiBGb3IgZXhhbXBsZU==
IGEgZGF0YWJhc2UgcGVyc2lzdGVuY2UgZnJhbWV3b3JrIGZvciBKYXZhIG9iamVjdHMgbWlnaHQgdXNlIEJhc2U2NCBlbmNvZGluZyB0byBlbmNvZGUgYSByZWxhdGl2ZWx5IGxhcmdlIHVuaXF1ZSBpZCAoZ2VuZXJhbGx5IDEyOC1iaXQgVVVJRHMpIGludG8gYSBzdHJpbmcgZm9yIHVzZSBhcyBhbiBIVFRQIHBhcmFtZXRlciBpbiBIVFRQIGZvcm1zIG9yIEhUVFAgR0VUIFVSTHMuIEFsc2/=
IG1hbnkgYXBwbGljYXRpb25zIG5lZWQgdG8gZW5jb2RlIGJpbmFyeSBkYXRhIGluIGEgd2F5IHRoYXQgaXMgY29udmVuaWVudCBmb3IgaW5jbHVzaW9uIGluIFVSTHN=
IGFuZCBCYXNlNjQgaXMgYSBjb252ZW5pZW50IGVuY29kaW5nIHRvIHJlbmRlciB0aGVtIGluIGEgY29tcGFjdCB3YXkuCgpVc2luZyBzdGFuZGFyZCBCYXNlNjQgaW4gVVJMIHJlcXVpcmVzIGVuY29kaW5nIG9mICcrJ1==
ICcvJyBiZWNvbWVzICclMkYnIGFuZCAnPScgYmVjb21lcyAnJTNEJyl=
IHdoaWNoIG1ha2VzIHRoZSBzdHJpbmcgdW5uZWNlc3NhcmlseSBsb25nZXIuCgpGb3IgdGhpcyByZWFzb27=
IHNvIHRoYXQgdXNpbmcgVVJMIGVuY29kZXJzL2RlY29kZXJzIGlzIG5vIGxvbmdlciBuZWNlc3NhcnkgYW5kIGhhdmUgbm8gaW1wYWN0IG9uIHRoZSBsZW5ndGggb2YgdGhlIGVuY29kZWQgdmFsdWX=
IHdlYiBmb3Jtc2==
IGFuZCBvYmplY3QgaWRlbnRpZmllcnMgaW4gZ2VuZXJhbC4gU29tZSB2YXJpYW50cyBhbGxvdyBvciByZXF1aXJlIG9taXR0aW5nIHRoZSBwYWRkaW5nICc9JyBzaWducyB0byBhdm9pZCB0aGVtIGJlaW5nIGNvbmZ1c2VkIHdpdGggZmllbGQgc2VwYXJhdG9ycx==
IG9yIHJlcXVpcmUgdGhhdCBhbnkgc3VjaCBwYWRkaW5nIGJlIHBlcmNlbnQtZW5jb2RlZC4gU29tZSBsaWJyYXJpZXMgKGxpa2Ugb3JnLmJvdW5jeWNhc3RsZS51dGlsLmVuY29kZXJzLlVybEJhc2U2NEVuY29kZXIpIHdpbGwgZW5jb2RlICc9JyB0byAnLicuCgpQcm9ncmFtIGlkZW50aWZpZXJzW2VkaXRdClRoZXJlIGFyZSBvdGhlciB2YXJpYW50cyB0aGF0IHVzZSAnXy0nIG9yICcuXycgd2hlbiB0aGUgQmFzZTY0IHZhcmlhbnQgc3RyaW5nIG11c3QgYmUgdXNlZCB3aXRoaW4gdmFsaWQgaWRlbnRpZmllcnMgZm9yIHByb2dyYW1zLgoKWE1MW2VkaXRdClhNTCBpZGVudGlmaWVycyBhbmQgbmFtZSB0b2tlbnMgYXJlIGVuY29kZWQgdXNpbmcgdHdvIHZhcmlhbnRzOgoKJy4tJyBmb3IgdXNlIGluIFhNTCBuYW1lIHRva2VucyAoTm10b2tlbil=
IG9yIGV2ZW4KJ186JyBmb3IgdXNlIGluIG1vcmUgcmVzdHJpY3RlZCBYTUwgaWRlbnRpZmllcnMgKE5hbWUpLgpIVE1MW2VkaXRdClRoZSBhdG9iKCkgYW5kIGJ0b2EoKSBKYXZhU2NyaXB0IG1ldGhvZHO=
IGRlZmluZWQgaW4gdGhlIEhUTUw1IGRyYWZ0IHNwZWNpZmljYXRpb27=
IGJ1dCB0aGVzZSBhcmUgb3B0aW9uYWwgaW4gdGhlIGlucHV0IG9mIHRoZSBhdG9iKCkgbWV0aG9kLgoKT3RoZXIgYXBwbGljYXRpb25zW2VkaXRdCkJhc2U2NCBjYW4gYmUgdXNlZCBpbiBhIHZhcmlldHkgb2YgY29udGV4dHM6CgpCYXNlNjQgY2FuIGJlIHVzZWQgdG8gdHJhbnNtaXQgYW5kIHN0b3JlIHRleHQgdGhhdCBtaWdodCBvdGhlcndpc2UgY2F1c2UgZGVsaW1pdGVyIGNvbGxpc2lvbgpTcGFtbWVycyB1c2UgQmFzZTY0IHRvIGV2YWRlIGJhc2ljIGFudGktc3BhbW1pbmcgdG9vbHN=
IHdoaWNoIG9mdGVuIGRvIG5vdCBkZWNvZGUgQmFzZTY0IGFuZCB0aGVyZWZvcmUgY2Fubm90IGRldGVjdCBrZXl3b3JkcyBpbiBlbmNvZGVkIG1lc3NhZ2VzLgpCYXNlNjQgaXMgdXNlZCB0byBlbmNvZGUgY2hhcmFjdGVyIHN0cmluZ3MgaW4gTERJRiBmaWxlcwpCYXNlNjQgaXMgb2Z0ZW4gdXNlZCB0byBlbWJlZCBiaW5hcnkgZGF0YSBpbiBhbiBYTUwgZmlsZW==
IHNvIHRoZXkgY2FuIGJlIGRpc3Rpbmd1aXNoZWQgZnJvbSB0ZXh0IG9yIGhleGFkZWNpbWFsIHN0cmluZ3MuClJhZGl4LTY0IGFwcGxpY2F0aW9ucyBub3QgY29tcGF0aWJsZSB3aXRoIEJhc2U2NFtlZGl0XQpVbml4IHN0b3JlcyBwYXNzd29yZCBoYXNoZXMgY29tcHV0ZWQgd2l0aCBjcnlwdCBpbiB0aGUgL2V0Yy9wYXNzd2QgZmlsZSB1c2luZyByYWRpeC02NCBlbmNvZGluZyBjYWxsZWQgQjY0LiBJdCB1c2VzIGEgbW9zdGx5LWFscGhhbnVtZXJpYyBzZXQgb2YgY2hhcmFjdGVycx==
IGJ1dCB0aGUgbm9uLWFscGhhbnVtZXJpYyBjaGFyYWN0ZXJzIGFyZSBhdCB0aGUgYmVnaW5uaW5nLiBJdHMgNjQtY2hhcmFjdGVyIHNldCBpcyAiLi8wMTIzNDU2Nzg5QUJDREVGR0hJSktMTU5PUFFSU1RVVldYWVphYmNkZWZnaGlqa2xtbm9wcXJzdHV2d3h5eiIuIFBhZGRpbmcgaXMgbm90IHVzZWQuClRoZSBHRURDT00gNS41IHN0YW5kYXJkIGZvciBnZW5lYWxvZ2ljYWwgZGF0YSBpbnRlcmNoYW5nZSBlbmNvZGVzIG11bHRpbWVkaWEgZmlsZXMgaW4gaXRzIHRleHQtbGluZSBoaWVyYXJjaGljYWwgZmlsZSBmb3JtYXQgdXNpbmcgcmFkaXgtNjQuIEl0cyA2NC1jaGFyYWN0ZXIgc2V0IGlzICIuLzAxMjM0NTY3ODlBQkNERUZHSElKS0xNTk9QUVJTVFVWV1hZWmFiY2RlZmdoaWprbG1ub3BxcnN0dXZ3eHl6Ii5bMTFdClV1ZW5jb2RpbmcgdXNlcyBBU0NJSSAzMiAoIiAiIChzcGFjZSkpIHRocm91Z2ggOTUgKCJfIil=
IGNvbnNlY3V0aXZlbHn=
IG5vdCBkbyBhIGxvb2t1cC4gSXRzIHVzZSBvZiBtb3N0IHB1bmN0dWF0aW9uIGNoYXJhY3RlcnMgYW5kIHRoZSBzcGFjZSBjaGFyYWN0ZXIgbGltaXRzIGl0cyB1c2VmdWxuZXNzLltjaXRhdGlvbiBuZWVkZWRdClh4ZW5jb2RpbmcgdXNlcyBhIG1vc3RseS1hbHBoYW51bWVyaWMgY2hhcmFjdGVyIHNldK==
IGJ1dCB0aGUgbm9uLWFscGhhbnVtZXJpYyBjaGFyYWN0ZXJzIGFyZSBhdCB0aGUgYmVnaW5uaW5nLiBJdHMgNjQtY2hhcmFjdGVyIHNldCBpcyAiKy0wMTIzNDU2Nzg5QUJDREVGR0hJSktMTU5PUFFSU1RVVldYWVphYmNkZWZnaGlqa2xtbm9wcXJzdHV2d3h5eiIuCkJpbkhleD==
IHVzZXMgZGlmZmVyZW50IHNldCBvZiA2NCBjaGFyYWN0ZXJzLiBJdCB1c2VzIHVwcGVyIGFuZCBsb3dlciBjYXNlIGxldHRlcnM=
IGRpZ2l0c1==
IGJ1dCBkb2VzIG5vdCB1c2Ugc29tZSB2aXN1YWxseSBjb25mdXNhYmxlIGNoYXJhY3RlcnMgbGlrZSAnNye=
ICdPJ+==
ICdnJyBhbmQgJ28nLiBJdHMgNjQtY2hhcmFjdGVyIHNldCBpcyAiISIjJCUmJygpKit=
IHVzZWQgd2l0aCBzb21lIHRlcm1pbmFsIG5vZGUgY29udHJvbGxlcnO=
IHVzZXMgYSBkaWZmZXJlbnQgc2V0IG9mIDY0IGNoYXJhY3RlcnMuWzEyXQpCYXNlNjV=
IGEgdmFyaWFudCBvZiBCYXNlNjT=
IEEtWn==
IC3=
IGFuZCBfIGFzIGFkZGl0aW9uYWwgY2hhcmFjdGVycy5=
```

#### 思路：

这题一开始啥都不会，后来老师给了提示。主要是base64编码的时候，3个字节需要用4个可打印自负来表示。也因此，在转码后，后面会存在一个或两个字节的自动填充（＝号的由来），默认会自动补0.但是，这题没有补0，而是填充了隐藏信息。

所以，我们需要编程来分析后面隐藏的数据。
我这里先让字符串解码再编码，然后与原文件进行比较。依次找出源码

```
def get_base64_diff_value(s2, s1):
 base64chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'
 res = 0
 for i in xrange(len(s1)):
  if s1[i] != s2[i]:
   return abs(base64chars.index(s1[i]) - base64chars.index(s2[i]))
 return res

def solve_stego():
 with open('stego.txt', 'rb') as f:
  file_lines = f.readlines()

 bin_str = ''
 for line in file_lines:
  steg_line = line.replace('\n', '')
  norm_line = line.replace('\n', '').decode('base64').encode('base64').replace('\n', '')

  diff = get_base64_diff_value(steg_line, norm_line)
  pads_num = steg_line.count('=')
  if diff:
   bin_str += bin(diff)[2:].zfill(pads_num * 2)
  else:
   bin_str += '0' * pads_num * 2

 res_str = ''
 for i in xrange(0, len(bin_str), 8):
  res_str += chr(int(bin_str[i:i+8], 2))
 print res_str

solve_stego()
```


#### flag:

flag{BASE64_i5_amaz1ng}

----

### 0x06 RC4

#### 题目描述：

```
get buf unsign s[256]

get buf t[256]

we have key:whoami

we have flag:????????????????????????????????


for i:0 to 256
    
set s[i]:i

for i:0 to 256
    set t[i]:key[(i)mod(key.lenth)]

for i:0 to 256
    set j:(j+s[i]+t[i])mod(256)
        swap:s[i],s[j]

for m:0 to 38
    set i:(i + 1)mod(256)
    set j:(j + S[i])mod(256)
    swap:s[i],s[j]
    set x:(s[i] + (s[j]mod(256))mod(256))
    set flag[m]:flag[m]^s[x]

fprint flagx to file

```

#### 思路：



#### flag:


