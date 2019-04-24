---
layout: post
category: 数据分析
title:  Datacon DNS攻击流量识别 内测笔记
catalog: true
excerpt:  3月下旬，抽空给DNS方向第一题进行了内部测试，将内部测试过程记录并分享给大家。
time: 2019.04.24 15:35:00
tags:
- Python
- 数据分析
- 大数据分析比赛

---

# Datacon-DNS攻击流量识别-内测笔记

Datacon-DNS方向题目一内测记录。

## 题目

![1](http://momomoxiaoxi.com/img/post/datacon/11.png)



数据下载链接：<https://804238.link.yunpan.360.cn/lk/surl_yF3jSPEC9QF>

解压密码：7K!y$V1+EJ.Yh]=4

内测时题目要求：

> **一、基础**DNS安全问题
>
> **题目简介：**选手在给定的数据包中通过分析发现围绕DNS展开的攻击等安全问题，赛题数据包大小为2.6G。
>
> **答案提交：**答案中需要包含攻击发起IP地址、攻击发起时间、攻击结束时间、攻击涉及域名列表等信息**。**



## 比赛结果

![WX20190424-164412@2x](http://momomoxiaoxi.com/img/post/datacon/WX20190424-164412@2x.png)



## 分析思路

考察DNS安全问题，因此首先寻找都有哪些DNS安全问题。

主要思路：

- 攻击者思路：搜索搜集对应的攻击类型，依据特征进行检测。
  - Google
  - Nmap dns攻击插件
  - Nessus dns攻击插件
- 防御者思路： 
  - 传统厂商检测规则
    - snort
    - suricata

## 困难与挑战

- pcap数据包较大，直接使用wireshark分析困难，卡顿现象特别严重，几乎不能分析。
  - 转用Tshark、Edipacp、snort、suricata进行分析
- 在无具体目标的情况下，进行全盘分析，会相对较累一些，很难覆盖所有常见攻击。主要在于未知有哪些攻击，围绕dns的攻击种类太多。
- 在背景流量的情况下，攻击可能只存在几秒钟，检测窗口会相对小一些。

## 时间线

1. 4小时手工+脚本分析+工具分析

   用于了解数据集，pcap切分、UDP、TCP比率、数据包占比、重传、DNS服务器IP、客户端IP等等

   ![WX20190307-232432](http://momomoxiaoxi.com/img/post/datacon/WX20190307-232432.png)

   ![WX20190307-232627](http://momomoxiaoxi.com/img/post/datacon/WX20190307-232627.png)

   ![WX20190307-232714](http://momomoxiaoxi.com/img/post/datacon/WX20190307-232714.png)

2. 4-8小时小时（学习Tshark、snort、suricata 规则，并对snort、suricata进行简单魔改，以达到适应该场景的需求）。

   1. snort、suricata为主机型IDS，其有内部IP网段（HOME_NET）的概念。

      - 所有query的目标ip=》映射为HOME_NET

        ```bash
        tshark -r q1_final.pcap -Y "dns.flags.response == 0" -T fields -e ip.dst |sort|uniq -c |awk -F ' ' '{if ($2>=0)print $2 }' > ip.dst
        ```

      - 请求IP为攻击外部IP

        ```bash
        tshark -r q1_final.pcap -Y "dns.flags.response == 1" -T fields -e ip.dst |sort |uniq -c |awk -F ' ' '{if ($2>=0)print $2 }' > ip.from
        ```

   2. 定义与修改snort规则，进行dns攻击检测

      如下：

      ```bash
      alert tcp $EXTERNAL_NET any -> $HOME_NET 53 (msg:"DNS zone transfer TCP"; flow:to_server,established; content:"|00 00 FC|"; offset:15; reference:arachnids,212; reference:cve,1999-0532; reference:nessus,10595; classtype:attempted-recon; sid:255; rev:13;)
      alert udp $EXTERNAL_NET any -> $HOME_NET 53 (msg:"DNS zone transfer UDP"; content:"|00 00 FC|"; offset:14; reference:arachnids,212; reference:cve,1999-0532; reference:nessus,10595; classtype:attempted-recon; sid:1948; rev:6;)
      ```

3. 具体分析各类攻击，进行检测（8小时+）

## 攻击与检测 

### DNS缓存投毒或网关劫持

- 恶意攻击者控制了你的网关，当你发送了一个查找`freebuf.com`的IP的请求的时候，中间人拦截住，并返回给你一个恶意网址的IP,你的浏览器就会把这个IP当做你想要访问的域名的IP!!这个IP是攻击者搭建的一个模仿了目标网站前端界面的界面，当你在该界面输入用户名密码或者付款操作的时候，就会中招。
- 检测逻辑：
  - 在非劫持情况下，在一定网络下，DNS的域名与IP地址的对应关系会在一定程度上保持稳定。如果发现在一定时间内，一大片域名的IP地址被集体篡改到某个IP，那么其便很有可能被劫持了。（考虑经过一段时间后，攻击暂停，IP又恢复到原始状态）
  - 全局统计domain和ip的对应关系，确认其稳定性。
- 检测结果：
  - 未发现攻击流量
    - 初步分析：
      - 用脚本过滤出domain和ip的对应关系，观察是否有抖动。结果为无抖动。
    - 可能检测模型有问题，或者无攻击（暂定为前者）

### DNS域传送漏洞

- 一般DNS区域传送操作只在网络里真的有备用域名DNS服务器时才有必要用到，但许多DNS服务器却被错误地配置成只要有client发出请求，就会向对方提供一个zone数据库的详细信息，即允许不受信任的因特网用户执行DNS区域传送操作。
  **危害:** 便于快速判断出某个特定区域的所有主机，获取域信息，如网络拓扑结构、服务器ip地址，为攻击者的入侵提供大量敏感信息。

- 攻击特征:

  - 攻击步骤:

    - 首先,查询一个域名的ns记录.
    - 然后,对权威服务器进行axfr或者ixfr查询

  - 区域传输攻击一般是AXFR或IXFR(AXFR最常见,为完全区域传送,IXFR为增量区域传送)

    > IXFR (incremental zone transfer  
    > request),是不同于传统的 AXFR (full zone transfer),IXFR 在做 Zone Transfer (**DNS** 的同步机制) 
    >  时,会以差异化的部份进行同步, 而 AXFR 则是以整个 Zone 进行同步

  - 一般数据以TCP传输。

- 检测逻辑:

  1. 简单判断是否存在dns-zone-transfer攻击

     ```bash
     tshark -n -r q1_final.pcap  -Y "((udp.port==53)||(tcp.port==53))"|grep -E 'AXFR|IXFR' > dns_zone_transfer
     ```

     确实存在攻击

     ![1](http://momomoxiaoxi.com/img/post/datacon/1.png)

  2. 调整过滤规则,取更详细数据

     - response中refuse的数据过滤掉
     - 正常攻击前,应该有一个ns查询
     - 具体AXFR的返回包具有很多域名数据(测试发现,攻击者只做了尝试?没有成功)

     ```bash
      tshark -r q1_tmp_00967_20190126002811.pcap -Y "dns.qry.type==252||dns.qry.type==251"
     ```

     攻击成功的尝试

     ```bash
     tshark -r q1_tmp_00967_20190126002811.pcap -Y "dns.qry.type==252||dns.qry.type==251" -2 -R "tcp.srcport==53" |grep  -v "Refused"
     ```

     发现其实整体没有攻击成功的流量，只有攻击尝试。

     攻击开始序列9474308-9678865

     ```bash
     tshark -r q1_final.pcap -Y "dns.qry.type==252||dns.qry.type==251" -T fields -e frame.number -e frame.time -e ip.src -e ip.dst -e dns.id -e dns.flags.response -e dns.qry.name -e dns.data  -e dns.flags.rcode > dns_zone_transfer_succ_end
     ```

  3. 检测结果

     - 攻击ip: 96.199.230.176
     - 开始时间:  Jan 26, 2019 00:22:34.008473000
     - 结束时间:  Jan 26, 2019 00:28:28.207297000
     - 影响域名列表:
       - 2667(实际没有攻击成功,因为无返回包)

     > 这种攻击不成功的dns-zone-transfer使用snort检测，可以自己写一个规则：
     >
     >  alert tcp $EXTERNAL_NET any -> $HOME_NET 53 (msg:"DNS zone transfer TCP"; content:"|00 00 FC|"; offset:14; reference:moxiaoxi; classtype:attempted-recon; sid:9999; rev:6;)
     >
     > snort自带的检测dns zone transfer，针对TCP会检测TCP连接状态（established），因此会把上述攻击过滤掉。
     >
     > 运行命令行：
     >
     > sudo snort -r q1_tmp_00967_20190126002811.pcap -c /etc/snort/snort.conf  -l ./log/ -A fast


### 非法DNS 动态更新：

> sudo suricata -c ./suricata.yaml -r q1_final.pcap -l ./log/
>
> 攻击由suricata检测出来，警报如下：
>
> ![WX20190309-142649](http://momomoxiaoxi.com/img/post/datacon/WX20190309-142649.png)
>
> alert udp $EXTERNAL_NET any -> $HOME_NET 53 (msg:"ET POLICY DNS Update From External net"; byte_test:1,!&,128,2; byte_test:1,!&,6      4,2; byte_test:1,&,32,2; byte_test:1,!&,16,2; byte_test:1,&,8,2; reference:url,doc.emergingthreats.net/2009702; classtype:policy-      violation; sid:2009702; rev:5; metadata:created_at 2010_07_30, updated_at 2010_07_30;)

- 在大多数生产环境中，系统管理员为DNS配置仅安全动态更新。通过仅允许可信域客户端计算机自动向DNS服务器注册自身，同时减少管理开销，显着降低了安全风险。但是，在某些情况下，管理员选择使用非Active Directory集成区域以保持与组织的策略兼容。此时，任何计算机都可以向DNS服务器发送注册请求。即使计算机不属于同一DNS域，DNS服务器也会自动在DNS数据库中添加请求计算机的记录。

- 检测逻辑，直接调用suricata，然后魔改一个输出插件即可得到结果。

- 检测结果：

  - 攻击者IP:

    ```
    19.220.251.87
    200.152.141.106
    237.205.156.233
    18.100.48.86
    ```

  - 攻击发起时间:2019-01-25T20:29:37.010896+0800-2019-01-25T23:49:27.633018+0800,2019-01-26T00:22:01.672532+0800-2019-01-26T00:50:11.665267+0800

  - 涉及域名列表: 共2024个域名

### DNS Amplification Attack 

- 放大攻击（也称为杠杆攻击，英文名字DNS Amplification Attack），利用回复包比请求包大的特点（放大流量），伪造请求包的源IP地址，将应答包引向被攻击的目标。对于DNS服务器来说，只需要抛弃不是自己发出去的请求包的应答包即可。从测试角度来说，向被测试服务器，发送大量的回复包，看是否被丢弃，同时观察此时的CPU利用率是否有急剧的上升即可。

- 攻击特征：

  - 通过递归查询从而放大流量，因此recursion=1，ANY参数。
  - 要求返回包远远大于发送包，一般返回包的要求大于2000。即dns.rr.udp_payload_size>=2000。

- 检测逻辑：

  魔改suricata DNS规则库，然后处理一波输出即可。

  ```bash
  alert udp any any -> $HOME_NET 53 (msg:"ET DOS DNS Amplification Attack Inbound"; content:"|01 00 00 01 00 00 00 00 00 01|"; dept      h:10; offset:2; pcre:"/^[^\x00]+?\x00/R"; content:"|00 ff 00 01 00 00 29|"; within:7; fast_pattern; byte_test:2,>,4095,0,relative      ; threshold: type both, track by_dst, seconds 60, count 5; classtype:bad-unknown; sid:2016016; rev:8; metadata:created_at 2012_12      _11, updated_at 2012_12_11;)
  ```

  suricata检测出的结果不全，可通过wireshark过滤

  ```
  Dns.flags.response==0 && dns.qry.type==255 && dns.rr.udp_payload_size>=2000 && dns.flags.recdesired==1
  ```

- 检测结果：

  ![WX20190309-212125](http://momomoxiaoxi.com/img/post/datacon/WX20190309-212125.png)

  检测出结果：

  175.222.102.169 → 188.141.167.218

  127.130.104.152 → 187.199.129.12

  105.191.150.205 → 70.85.232.160

  > 这里要忽略那些反射放大长度过大的攻击请求，以及refused的数据包。



### DNS子域名枚举

> 检测比较粗略，结果估计还需要进一步分析。

1. DNS子域名枚举主要用于暴力枚举子域名信息，方便后期进行深入攻击。

2. 检测逻辑：
   1. 检测所有domain信息，观察是否有对某个域名的枚举现象。而且枚举现象在时间内进行。
   2. 提取了所有domain信息，明显有子域名枚举攻击。
   3. 短时间内同一个ip对一个domain进行枚举

3. 检测结果：

   1. 攻击者IP:

      144.202.64.226

### DNS tunnel 

1. DNS Tunneling，是隐蔽信道的一种，通过将其他协议封装在DNS协议中传输建立通信。因为在我们的网络世界中DNS是一个必不可少的服务，所以大部分防火墙和入侵检测设备很少会过滤DNS流量，这就给DNS作为一种隐蔽信道提供了条件，从而可以利用它实现诸如远程控制，文件传输等操作，现在越来越多的研究证明DNS Tunneling也经常在僵尸网络和APT攻击中扮演着重要的角色。

2. 检测逻辑：
   - 提取了所有的domain信息,查看domain信息。隧道在domain有明显特征。
3. 检测结果：
   - 基本无DNS tunnel信息。



### Nsec 枚举

> 给答案之前没发现的类型。

- 这个攻击可以参考之前写的博客：http://momomoxiaoxi.com/2017/11/05/EISCTF/#214--nsec%E8%AE%B0%E5%BD%95

  简单来讲，就是攻击者利用NSEC能获得下一条数据的信息，来重复获取下一个dns记录，从而达到泄漏的作用。

- 检测逻辑：

  - 获得所有Nsec类型的DNS记录

  - 攻击者IP、攻击发起时间、枚举使用域名列表

  - ```bash
     tshark -r q1_final.pcap   -Y '(dns.resp.type == 47 and dns.flags.response == 1)' -T fields -e frame.number -e frame.time -e ip.dst  -e ip.src -e dns.qry.name -e dns.resp.type -e dns.resp.name
    ```

- 检测结果：

  - 攻击者IP：175.222.102.169（失败）、105.191.150.205（失败）、6.116.183.244（成功）
  - 时间：Jan 25, 2019 22:30:34.573890000 CST 
  - 域名列表： d1a4.cc（失败）e24561.com.cn（成功）

  



## 收获

1. 整个题目出得还是非常有意义，包含了很多常见的DNS攻击。从数据集可以明显看出，数据准备还是花了很长时间，尤其在设计出题点、敏感信息处理、背景流量预设等等地方，会耗费较多精力。这里对各位出题人大大表示感谢！！！

2. 在整个分析过程中，学到了很多，值得记录。

   - 大数据包Pcap的分析与处理。较大的Pcap包将无法通过wireshark直接分析，需要借助其他命令行工具进行可视化分析。
   - 通过统计工具分析整体Pcap包流量状况，TCP/UDP占比、重传率、解析服务器IP统计、规则过滤器等等。通过各类统计分析，大约估计整体流量状况。
   - 细化DNS攻击分析，更细化地了解了各类DNS攻击的攻击方式、检测方式，并通过此次机会学习了一波Snort和Suricat的使用。

3. 看了后续大家提交的WP，还是看到了很多有趣的思路。

   1. 大机器直接wireshark分析；
   2. 专业数据清洗流；
   3. 通过时频图进行分析；
   4. 通过每日评分系统，侧信道确定答案；
   5. ….

   等待大家后续自行分享了，这里就不再叙述。

4. 希望大家在这场比赛中有所收获！：）



## 参考

1. https://www.securitynik.com/2014/05/analyzing-dns-zone-transfer-both.html
2. https://github.com/eldondev/Snort/blob/master/rules/dns.rules
3. https://www.securityforrealpeople.com/2015/01/detecting-malware-through-dns-queries.html
4. https://www.cnblogs.com/liun1994/p/6142505.html
5. https://github.com/pevma/rule2alert
6. http://www.kaiyuanba.cn/content/network/snort/Snortman.htm
7. http://www.tutorialspoint.com/articles/configuring-dns-server-for-secure-only-dynamic-updates
8. https://social.technet.microsoft.com/Forums/ie/en-US/989e0771-1d6f-4711-bfce-f082ce77b5d9/dns-secured-v-insecure-dynamic-updates?forum=winserverDS
9. http://blog.sina.com.cn/s/blog_90bb1f200101iazl.html
10. https://yq.aliyun.com/articles/396578/
11. https://eprint.iacr.org/2010/115.pdf
12. https://www.researchgate.net/publication/221655355_A_Security_Evaluation_of_DNSSEC_with_NSEC3
13. https://docs.infoblox.com/display/NAG8/Detecting+NXDOMAIN+Attacks