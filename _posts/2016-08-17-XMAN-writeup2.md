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

## 课上练习

------


### CM1

[题目附件](http://xman.xctf.org.cn/media/task/cdaf8d32-a95c-4622-b4fb-f599c56ea9e4.exe)

下载下来，拉进IDA，然后F5，可以看到源码

![image](http://momomoxiaoxi.com/img/post/XMAN/2.png)

分析下源码，就是一个字符串验证，然后查看下Str2的值，为Thi5_i5_T0o_E4sy
输入下就得到了下面的答案

![image](http://momomoxiaoxi.com/img/post/XMAN/3.png)


------

### CM2

丢进IDA

```
int __cdecl main(int argc, const char argv, const char envp)
{
  int v3;  edx@1
  char v5[9];  [sp+Ch] [bp-3Ch]@1
  int v6;  [sp+15h] [bp-33h]@1
  int v7;  [sp+19h] [bp-2Fh]@1
  __int16 v8;  [sp+1Dh] [bp-2Bh]@1
  char v9;  [sp+1Fh] [bp-29h]@1
  char v10;  [sp+20h] [bp-28h]@1
  char v11;  [sp+21h] [bp-27h]@1
  __int16 v12;  [sp+45h] [bp-3h]@1
  char v13;  [sp+47h] [bp-1h]@1

  (_DWORD )&v5[1] = 0;
  (_DWORD )&v5[5] = 0;
  v6 = 0;
  v7 = 0;
  v5[0] = 0;
  v8 = 0;
  v10 = 0;
  v9 = 0;
  memset(&v11, 0, 0x24u);
  v12 = 0;
  v13 = 0;
  printf(Format);
  scanf(a16s, v5);
  sub_401000(v3, &v10, (int)v5, strlen(v5));      base64
  if ( !memcmp(&v10, aQjrzzty0x2k1x2, 0x19u) )
    printf(aXmanS, v5);
  else
    printf(aPleaseReverseM);
  system(Command);
  return 0;
}
```

分析下sub_401000，发现逻辑很复杂。
然后，尝试解一下aQjrzzty0x2k1x2对应的字符串，因为看起来像base64

.data:00403010 aQjrzzty0x2k1x2 db 'QjRzZTY0X2k1X2MwbW1vbg==',0 

解码出来B4se64_i5_c0mmon

猜测这就是flag，输入下。果然对了！

```
Input Key:B4se64_i5_c0mmon
xman{B4se64_i5_c0mmon}
请按任意键继续. . .



```


------

### CM4

拉入IDA，如常，F5得到源码：

![image](http://momomoxiaoxi.com/img/post/XMAN/4.png)
 
 ```
  printf(Format);
  scanf(a16s, &v4);
  sub_401100(&v18);
  sub_401130(&v18, &v4, strlen(&v4));
  sub_4011E0((int)&v11, (int)&v18);
  if ( !memcmp(&v11, &unk_403010, 0x11u) )
    printf(aMd5IsGoodXmanS, &v4);
  else
    printf(aPleaseReverseM);
  system(Command);
 ```
 显然，我们可以推测这里应该用到了md5.依据上面的题的思路，我们首先猜一下这里可能就是做了一个md5加密。我们找到比较的字符串是什么。
 
 
 ```
 .data:00403010 unk_403010      db  7Fh ;              ; DATA XREF: _main+B4o
.data:00403011                 db 0EFh ; 
.data:00403012                 db  61h ; a
.data:00403013                 db  71h ; q
.data:00403014                 db  46h ; F
.data:00403015                 db  9Eh ; 
.data:00403016                 db  80h ; €
.data:00403017                 db 0D3h ; 
.data:00403018                 db  2Ch ; ,
.data:00403019                 db    5
.data:0040301A                 db  59h ; Y
.data:0040301B                 db 0F8h ; 
.data:0040301C                 db  8Bh ; 
.data:0040301D                 db  37h ; 7
.data:0040301E                 db  72h ; r
.data:0040301F                 db  45h ; E
.data:00403020                 db    0
.data:00403021                 db    0
.data:00403022                 db    0
```

解码下，
7FEF6171469E80D32C0559F88B377245

![image](http://momomoxiaoxi.com/img/post/XMAN/5.png)


```
Input Key:admin888
md5 is good! xman{admin888}
请按任意键继续. . .

```


-----

### CM5

首先发现这是一个RC4加密的代码，密钥是I_am_not_the_flag，密文是6EDCE5E7B7CF3CC581E7D0B5CBAA91DE。
然后，就是一个反解，可以得到密码是rc4_i5_v3ry_e4sy

------

### CM6