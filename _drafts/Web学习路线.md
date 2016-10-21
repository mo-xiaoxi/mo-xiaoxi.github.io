# Web学习路线

## 0x00 前言

> 这是我依据我浅薄的学习经历来总结的，难免在里面有很多缺漏甚至谬误的地方。一来是因为我个人能力的局限性，二来是Web安全这方面的知识点实在太多太杂，很难完全总结下来。所以，如果你发现有什么问题，请在下面评论或直接发email给我，我好修改，谢谢大家。😄

去年，我与我的团队在我母校创立了一个安全协会与对应的实验室，我们想通过一个这样的学生组织为一些对信息安全知识有充足兴趣的人提供一个较好的学习平台，让大家能够更加容易地跨入信息安全的大门。

之前，我好像过于强调个人自学能力的重要性，而有些忽视对大家的系统培训，只是从一些角度来告诉大家，哪些东西需要去学，安全方面有哪些东西。

现在，我发现这个做法可能不大妥当，安全范围实在太广，让大家自己直接去学，可能大家很难去完全把控好学习的流程，哪些先学，哪些后学。这样，很有可能就大大降低了大家的学习效率。所以，我打算写一篇类似学习路线的博文，希望能给大家一些帮助。

此外，由于我先前学习的主要是Web方向，所以这篇博文也只是关于Web安全方向的学习流程。后面，我也会尽量请求一些老司机们能花一些时间为大家写一些其他方向的入门学习路线。



> 一些学习建议：
>
> 	1. 想要了解某个技术，请阅读一些相关文档；想要明白某个技术，请尝试实践这些技术；想要悟透某个技术，请尝试与他人讲解这个技术。
> 	2. 学习一个新的知识点，请务必做学习笔记。如果你有博客，可以尝试写一些技术博文，不需要担心学习笔记太低端，不符合博文的逼格。
> 	3. 想要快速了解一个领域的前沿情况，请查阅该领域的近几年的优秀论文、看该领域最优秀的人在研究什么（当然最好的方式是找优秀的大牛直接咨询，前提你认识哪些大牛）



## 0x01 基础语言

1. 首先，花4天时间大略过一下最基础的语言语法HTML／CSS、JavaScript、PHP、SQL的基本知识（平均一个语言4小时足矣），要做到看到这些代码的时候不会畏惧，能看懂大部分，看到不懂的能通过google或者相关文档查懂。主要参考：http://www.w3school.com.cn/  http://www.runoob.com/

2. 现在，你初步了解了一些语言知识，接下来你就需要开始在实践中学习与巩固。

   > 我一直有一个这样的观点，其实我们学习一样东西，最好的学习方式就是实践与理论双螺旋式学习，先学习一部分东西，然后折腾实践，遇到不懂的，再去查，查懂了，再去实践...周而复始，一步步学习。最后，再通过一本相对系统的书，对该知识从头复习一遍。这样，就能基本掌握这部分知识。

   这里，我建议你搭建一个个人的博客，初步了解这些语言的应用。在搭建博客的过程中，你需要学习对应的linux操作、Apache+PHP+Mysql环境的配置、域名解析、服务器端口配置等等。如何搭建博客，请自行google搜索。

   此外，你也可以编写一些好玩的网页来学习这些知识点。

   第2步大约花费你7-14天的时间。

3. 如果经过第2步，你掌握了以下几个技能就可以进入下一个阶段了。

   -  大体能看到百分之80以上简单的HTML／CSS、JavaScript、PHP、SQL的代码
   -  基本了解一个index.php是如何解析的。（一个包含了PHP、SQL字符串、JavaScript、HTML／CSS代码的文件）明白为什么前端看不到PHP，明白为什么后端PHP中可以混杂其他的语言
   -  能基本在Linux下配置Apache+PHP+Mysql环境（请尽量使用最原生的Apache、Mysql，不要使用集成工具！尽量使用命令行，不要使用Gui）
   -  能基本使用FTP工具、SSH

4. 后期，你还需要了解DOM、BOM、ajax、json的知识。

浏览器插件与技巧
高效与舒适的上网环境需要优秀的浏览器和众多插件支持。
Chrome/Firefox两者选其一，并研究其开发者工具的功能，参见[Chrome 浏览器开发者工具](http://www.cnblogs.com/QLeelulu/archive/2011/08/28/2156402.html)；Chrome/Firefox快捷键和代理设置；
插件的使用：Firebug、Hackbar、Tamper Data、Adblock、SwitchySharp；[从输入 URL 到页面加载完成的过程中都发生了什么事情](http://fex.baidu.com/blog/2014/05/what-happen/)；
[网页编码就是那点事](http://www.qianxingzhem.com/post-1499.html)。

## 0x02 其它基础

1. 了解最基本的C/S物理结构

    ![1](/Users/moxiaoxi/Desktop/1.png)

2. 了解基本的HTTP协议相关知识

   - 掌握HTTP请求：GET、POST、HEAD，能基本读懂一个HTTP数据包，了解GET、POST、HEAD的区别

   - 能看懂HTTP响应包 404 403 200 500 ...

   - 了解Cookie、session、token，知道是什么东西，作用是什么，为什么需要这些东西

   - 了解Referer、X-Frame-Options等等的作用

   参考：

   1. http://www.ruanyifeng.com/blog/2016/08/http.html
   2. http://www.cnblogs.com/li0803/archive/2008/11/03/1324746.html
   3. http://momomoxiaoxi.com/2016/01/26/NetworkAnalysis/

3. 了解基本的浏览器解析的流程与原理

   参考：

   1. http://fex.baidu.com/blog/2014/05/what-happen/
   2. http://ued.ctrip.com/blog/how-browsers-work-rendering-engine-html-parsing-series-ii.html
   3. http://www.jianshu.com/p/e305ace24ddf
   4. http://www.cnblogs.com/yuezk/archive/2013/01/11/2855698.html

   这一部分可能会比较复杂，大家最开始只需要有一种基本的概念即可，后面逐渐弄懂他们。

4. 了解一些编码

   1. 宽字节编码 了解一下http://www.cnbraid.com/2016/02/28/sql4/

   2. HTML字符实体编码、对应的命名实体、URL编码、JS编码、CSS编码、Base64编码

      这三种编码在挖掘XSS漏洞的时候尤为有用！当然，有时候其他的注入攻击的时候也会用到

      参考：

      	1. http://www.ruanyifeng.com/blog/2010/02/url_encoding.html
      	2. https://security.yirendai.com/news/share/26
      	3. http://www.freebuf.com/articles/web/43285.html
      	4. http://bobao.360.cn/learning/detail/292.html（比较推荐这个）
      	5. http://paper.seebug.org/papers/Archive/drops2/XSS%E4%B8%8E%E5%AD%97%E7%AC%A6%E7%BC%96%E7%A0%81%E7%9A%84%E9%82%A3%E4%BA%9B%E4%BA%8B%E5%84%BF%20---%E7%A7%91%E6%99%AE%E6%96%87.html（还有这个）
      	6. 对应的编解码工具：http://evilcos.me/lab/xssor/   http://evilcos.me/lab/xssee/

   3. 序列化编码问题

      Java、PHP、Python序列化问题

      1. http://www.crazydb.com/archive/Java%E3%80%81PHP%E3%80%81Python%E7%9A%84%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E9%97%AE%E9%A2%98
      2. http://www.hollischuang.com/archives/1140
      3. http://www.wtoutiao.com/p/1e1gMC1.html
      4. http://www.milw0rm.cn/Article/web/20161020/607.html

      编码安全问题在前端安全方面一直是一个很重要的方面，请大家务必重视。不过这个方面一开始很难完全掌握，大家一开始只需要有一个较为完整的概念即可，后面迭代深入学习。

5. 稍微了解下同源策略

   http://www.ruanyifeng.com/blog/2016/04/same-origin-policy.html

   知道为何需要同源策略，同源策略有什么好处与坏处，同源策略主要是来应对什么类型的攻击。



在这一块的学习过程中，你肯定会遇到很多很多不懂的地方，也会感觉学习这个东西很难。不过，你要放心，这是最正常的现象。因为我把一些后面的东西都放在了这里。大家可以前后花一周时间，把第二块过下来。每天学习一个知识点，并写一篇博文。然后，再花3-5天的时间，巩固复习这一块。



## 0x02 基本攻击知识普及

### SQL注入

1. 最基础的注入（数字型、字符型、搜索型）注入模式：基于布尔 基于时间 基于报错 联合注入 堆查询注入

2. 宽字节注入

3. 二次注入

4. http头注入

5. Nosql注入

6. 偏移注入

7. 各种绕waf注入

   ​

有时候，工具或者单纯手注会比较麻烦，此时你需要用Python或者定制化sqlmap来进行注入：）

参考：

1. https://github.com/Audi-1/sqli-labs （首要推荐，刷完这些题基本就入门了！）
2. http://websec.ca/kb/sql_injection （基础知识）	
3. http://www.cnblogs.com/lcamry/articles/5625276.html（宽字节）
4. http://ecma.io/?tag=%E4%BA%8C%E6%AC%A1%E6%B3%A8%E5%85%A5（二次注入）
5. ​





MySQL MSSQL Oracle PostgreSQL Access SQLite。

| XXE  | 介绍XXE原理以及利用方式 |
| ---- | ------------- |
|      |               |

sqlmap

发了Paper CVE 写了相关工具 有分享

[https://github.com/Audi-1/sqli-labs](https://github.com/Audi-1/sqli-labs) [![User Link](http://blog.knownsec.com/Knownsec_RD_Checklist/v3.1.html_files//ilink.png)](https://github.com/Audi-1/sqli-labs)

SQLI-LABS is a platform to learn SQLI



### XSS攻击



-   - 内部平台：ks-xsslab_open
    - 可以上手
      - XSS
      - CSRF
      - ClickJacking
-   [http://xss-quiz.int21h.jp/](http://xss-quiz.int21h.jp/) [![User Link](http://blog.knownsec.com/Knownsec_RD_Checklist/v3.1.html_files//ilink.png)](http://xss-quiz.int21h.jp/)
    - ![attach](http://blog.knownsec.com/Knownsec_RD_Checklist/v3.1.html_files//icons/attach.png) [答案：xss_quiz.txt](http://blog.knownsec.com/Knownsec_RD_Checklist/res/xss_quiz.txt) [![User Link](http://blog.knownsec.com/Knownsec_RD_Checklist/v3.1.html_files//ilink.png)](http://blog.knownsec.com/Knownsec_RD_Checklist/res/xss_quiz.txt)
-   [http://prompt.ml/0](http://prompt.ml/0) [![User Link](http://blog.knownsec.com/Knownsec_RD_Checklist/v3.1.html_files//ilink.png)](http://prompt.ml/0)
    - [答案：https://github.com/cure53/XSSChallengeWiki/wiki/prompt.ml](https://github.com/cure53/XSSChallengeWiki/wiki/prompt.ml) [![User Link](http://blog.knownsec.com/Knownsec_RD_Checklist/v3.1.html_files//ilink.png)](https://github.com/cure53/XSSChallengeWiki/wiki/prompt.ml)
-   [http://escape.alf.nu/](http://escape.alf.nu/) [![User Link](http://blog.knownsec.com/Knownsec_RD_Checklist/v3.1.html_files//ilink.png)](http://escape.alf.nu/)
    - [答案：http://blog.nsfocus.net/alert1-to-win-write-up/](http://blog.nsfocus.net/alert1-to-win-write-up/) [![User Link](http://blog.knownsec.com/Knownsec_RD_Checklist/v3.1.html_files//ilink.png)](http://blog.nsfocus.net/alert1-to-win-write-up/)

    深入XSS（XSS Rootkit、HTML富文本、flash逆向）

### 文件上传

### CSRF

### 远程加载

### 乌云

### PHP审计

### 旁注

### 提权



### ldapxpathxml

溢出提权、第三方组件提权、虚拟主机提权等技术

### 数据库备份恢复

### 一句话木马

### 内网渗透

### 先进技术

科普 `XSIO`、`Location-Spoofing`、`DNS-Amplification` 等不为人们熟悉的猥琐流攻击手段

高级专题

                云架构安全（针对大型网站）

                大数据安全（nosql啥啥的）

                隐私安全（cookie、日志、脱裤、http、云端数据）

                WAF绕过







然后就是数据库了～mysql sqlserver 随便先学一个～前期学会 selsct 就行，结合php尝试自己写一个查询数据库的脚本 ～来了解sql注入的原理～这样进步会很快……

Iis 解析漏洞
nginx 解析漏洞
tomcat 后台上传
jboss

包含漏洞

代码执行漏洞

`XML注入`、`JSON注入`、Web Server远程部署漏洞等漏洞。



### 密码学基础

### 源码泄漏

代码托管搜索

## 0x03 一些常用工具学习与使用

###  Hackbar

###  Burpsuite 

###  Sqlmap

###  Nosql

###  Metasploit

###  Chrome 

chopper

Nessus

- HTTP抓包与调试
  - Firefox插件
    - Firebug
      - 抓包与各种调试
    - Tamper Data
      - 拦截修改
    - Live Http Header
      - 重放功能
    - Hackbar
      - 编码解码/POST提交
    - Modify Headers
      - 修改头部
  - Fiddler
    - 浏览器代理神器
    - 拦截请求或响应
    - 抓包
    - 重放
    - 模拟请求
    - 编码解码
    - 第三方扩展
      - Watcher
        - Web前端安全的自动审计工具
  - Wireshark
    - 各种强大的过滤器语法
  - Tcpdump
    - 命令行的类Wireshark抓包神器

- - 《JavaScript DOM编程艺术》

- 了解DOM

  - 这同样是搞好前端安全的必要基础

- 库

  - jQuery

    - 优秀的插件应该体验一遍，并做些尝试
    - 官方文档得过一遍

  - D3.js

  - ECharts

    - 来自百度

  - Google API

  - ZoomEye Map组件

    - ZoomEye团队自己基于开源的打造

  - AngularJS

    - Google出品的颠覆性前端框架

  - Bootstrap

    - 应该使用一遍

    http://www.ruanyifeng.com/blog/2012/05/internet_protocol_suite_part_i.html


- Firefox
  - Firebug
    - 调试JavaScript，HTTP请求响应观察，Cookie，DOM树观察等
  - Tamper Data
    - 拦截修改
  - Live Http Header
    - 重放功能
  - Hackbar
    - 编码解码/POST提交
  - Modify Headers
    - 修改头部
  - GreaseMonkey
    - [Original Cookie Injector for Greasemonkey](http://userscripts.org/scripts/show/119798) [![User Link](http://blog.knownsec.com/Knownsec_RD_Checklist/v3.1.html_files//ilink.png)](http://userscripts.org/scripts/show/119798)
  - NoScript
    - 进行一些JavaScript的阻断
  - AutoProxy
    - 翻墙必备
- Chrome
  - F12
    - 打开开发者工具，功能==Firebug+本地存储观察等
  - SwichySharp
    - 翻墙必备
  - CookieHacker
    - [http://evilcos.me/?p=366](http://evilcos.me/?p=366) [![User Link](http://blog.knownsec.com/Knownsec_RD_Checklist/v3.1.html_files//ilink.png)](http://evilcos.me/?p=366)
- Web2.0 Hacking
  - XSS'OR
    - 常用其中加解密与代码生成
    - [http://evilcos.me/lab/xssor/](http://evilcos.me/lab/xssor/) [![User Link](http://blog.knownsec.com/Knownsec_RD_Checklist/v3.1.html_files//ilink.png)](http://evilcos.me/lab/xssor/)
    - [源码：https://github.com/evilcos/xssor](https://github.com/evilcos/xssor) [![User Link](http://blog.knownsec.com/Knownsec_RD_Checklist/v3.1.html_files//ilink.png)](https://github.com/evilcos/xssor)
  - XSSEE 3.0 Beta
    - Monyer开发的，加解密最好用神器
    - [http://evilcos.me/lab/xssee/](http://evilcos.me/lab/xssee/) [![User Link](http://blog.knownsec.com/Knownsec_RD_Checklist/v3.1.html_files//ilink.png)](http://evilcos.me/lab/xssee/)
  - Online JavaScript beautifier
    - JavaScript美化工具，分析JavaScript常用
    - [http://jsbeautifier.org/](http://jsbeautifier.org/) [![User Link](http://blog.knownsec.com/Knownsec_RD_Checklist/v3.1.html_files//ilink.png)](http://jsbeautifier.org/)
  - BeEF
    - The Browser Exploitation Framework
    - [http://beefproject.com/](http://beefproject.com/) [![User Link](http://blog.knownsec.com/Knownsec_RD_Checklist/v3.1.html_files//ilink.png)](http://beefproject.com/)
- HTTP代理
  - Fiddler
    - 非常经典好用的Web调试代理工具
  - Burp Suite
    - 神器，不仅HTTP代理，还有爬虫、漏洞扫描、渗透、爆破等功能
  - mitmproxy
    - Python写的，基于这个框架写神器实在太方便了

  ​
- 漏洞利用
  - sqlmap
    - SQL注入利用最牛神器，没有之一
  - Metasploit
    - 最经典的渗透框架
  - Hydra
    - 爆破必备
- 抓包工具
  - Wireshark
    - 抓包必备
  - Tcpdump
    - Linux下命令行抓包，结果可以给Wireshark分析
- Seebug + ZoomEye
  - 类似这类平台都是我们需要的
  - Seebug类似的
    - https://www.exploit-db.com/
  - ZoomEye类似的
    - https://www.shodan.io/

Firefox

浏览器模式http://www.ruanyifeng.com/blog/2015/02/mvcmvp_mvvm.html



用浏览器的工具查看HTTP缓存实现，看看不同种类的网站，不同类型的资源，具体是如何控制缓存的；也可以自己用Python写一个简单的HTTP服务器，模拟返回各种HTTP Headers来观察浏览器的反应。

 http://prompt.ml/0



http://redtiger.labs.overthewire.org/

1. ​

    



**Firefox下**

1. Firebug，调试js，HTTP请求响应观察，Cookie，DOM树观察等；
2. GreaseMonkey，自己改了个Cookie修改脚本，其他同学可以用这款：[Original Cookie Injector for Greasemonkey**](http://userscripts.org/scripts/show/119798)；
3. Noscript，进行一些js的阻断；
4. AutoProxy，翻墙必备；

**Chrome下**

1. F12打开开发者工具，功能==Firebug+本地存储观察等；
2. SwichySharp，翻墙必备；
3. Cookie修改脚本，自己写了一个Chrome扩展（已开源：[Cookie利用神器：CookieHacker](http://evilcos.me/?p=366)），其他同学可以自己到Chrome扩展搜个好用的；

**前端渗透工具**

1. [XSS’OR**](http://evilcos.me/lab/xssor/)，我开发的，常用其中加解密与代码生成，源码放到了这：[evilcos/xssor · GitHub**](https://github.com/evilcos/xssor)；
2. [XSSEE 3.0 Beta**](http://evilcos.me/lab/xssee/)，Monyer开发的，加解密最好用神器；
3. [Online JavaScript beautifier**](http://jsbeautifier.org/)，js美化工具，分析js常用；
4. 前端攻击框架，推荐BeEF及一些小伙伴开发的XSS盲打工具，我自己也有款，不过不轻易示人；
5. 编码环境
   - pip
   - Docker/Vagrant
   - tmux/screen
   - vim
   - Markdown
   - zsh + oh-my-zsh
   - Python2.7+/3+
   - \>Django1.4
     - [http://djangobook.py3k.cn/2.0/](http://djangobook.py3k.cn/2.0/) [![User Link](http://blog.knownsec.com/Knownsec_RD_Checklist/v3.1.html_files//ilink.png)](http://djangobook.py3k.cn/2.0/)
     - [Django Debug Toolbar](http://django-debug-toolbar.readthedocs.org/en/latest/) [![User Link](http://blog.knownsec.com/Knownsec_RD_Checklist/v3.1.html_files//ilink.png)](http://django-debug-toolbar.readthedocs.org/en/latest/)
     - 其他框架
       - web.py
       - Flask
       - Tornado
   - node.js
   - Ubuntu/Gentoo/Centos
   - ipython
   - 版本控制
     - 废弃SVN，全面拥抱Git
     - GitLab
   - Nginx+uWSGI

**HTTP代理工具**

1. Fiddler，即可，不用再寻找其他的了，其中的watcher插件可以玩玩，找漏洞的；
2. Burp Suite，神器，不仅HTTP代理，还有爬虫、漏洞扫描、渗透、爆破等功能；

**漏洞扫描工具**

1. AWVS，不仅漏扫方便，自带的一些小工具也好用；
2. Python自写脚本/工具，好漏洞是你用AWVS等就能发现的？洗洗睡吧；
3. Nmap，绝对不仅仅是端口扫描！几百个脚本；

**漏洞利用**

1. sqlmap，SQL注入利用最牛神器，没有之一；
2. Metasploit，主机渗透框架，而Web层面上的就是知道创宇里的一些好玩意了（我可能在吹牛）；
3. 一些社工平台，好的都匿了；
4. Hydra，爆破必备；

**抓包工具**

1. Wireshark，抓包必备；
2. tcpdump，Linux下命令行抓包，结果可以给Wireshark分析；

**大数据平台**

1. ZoomEye，知道创宇开放的一个网络空间搜索引擎，搜搜组件就知道：[ZoomEye（钟馗之眼）**](http://www.zoomeye.org/)，可以认为我在广告；
2. SHODAN， 老外开放的一个网络空间搜索引擎，搜搜主机设备就知道：[SHODAN – Computer Search Engine；**](http://www.shodanhq.com/)
3. Google，：）

Linux 鸟哥的私房菜

- 《鸟哥的Linux私房菜》
- 《Linux Shell脚本攻略》
- 《UNIX编程艺术》
- 《Software Design 中文版 01》《Software Design 中文版 02》《Software Design 中文版 03》

VIM

正则表达式

内网渗透

### 0x04 持续学习



### 积累一些自己的小工具

1. 信息泄漏
2. 盲注脚本
3. 验证码识别
4. pytesser

平时训练、项目、比赛、如何协调 课程

- 理池
  - 爬虫「稳定」需要
- 网络请求
  - wget/curl
  - urllib2/httplib2/requests
  - ![idea](http://blog.knownsec.com/Knownsec_RD_Checklist/v3.1.html_files//icons/idea.png) scrapy
- 验证码破解
  - pytesser


- 数据结构

  - JSON
  - cPickle
  - protobuf

- 数据存储及处理

  - 数据库

    - MySQL
    - MongoDB
    - Cassandra
    - Hadoop体系
    - Redis
    - Sqlite
    - bsddb
    - ElasticSearch

  - 大数据处理

    - Hive

    - Spark

    - ELK

      - ElasticSearch
      - Logstash
      - Kibana

      https://pentesterlab.com/

## 0x05 提高方向

代码审计。熟悉各类框架

## 0x06 未来

### 科研界

### 工业界

csp

https

CDN 

http2.0



## 0x07 一些有趣的Web项目



1. Seebug 大数据

2. zoomeye 重写

3. 自主设计扫描器

4. 威胁情报

5. # Anti-Phishing

   [![img](https://pic2.zhimg.com/a33a4b73ea7991badf4675383447f225_m.jpg)bsdr ](https://www.zhihu.com/people/bsdr)· 2 天前

   Anti-Phishing是一个为反钓鱼而发起的项目,旨在通过无状态扫描扫描全世界范围内的网站,并识别出其中的钓鱼网站,以此为基础进行反钓鱼.

   事情的起因见[这一次，我不服 - Python Hacker - 知乎专栏](https://zhuanlan.zhihu.com/p/22289520)，最近看到一个问题：[http://www.zhihu.com/question/51198978](http://www.zhihu.com/question/51198978)，安全领域最大的悲哀和无奈或许就在于这里：当越来越多的安全厂商开始宣扬大数据、云安全、威胁情报、态势感知的时候，还有无数普通人因为各种各样的钓鱼、电信欺诈、信息泄漏、恶意APP而上当受骗，尽管这些东西在专业人员的眼中再简单不过。并不是吐槽公众缺乏安全意识，而是有诚意为公众服务的人太少了。

   至于项目，目前已经基本完成第一阶段,编写了一套类似ZoomEye的分布式无状态的扫描器,正在服务器上调试和部署.希望做一些力所能及的小事。

   **Be a hacker，be a hero。**





## 思路

1.专注
​    我觉得在学习网络安全和任何东西的时候都要分阶段专注学习、
​    比如研究xss 那我这一个月就什么也不做只研究xss 把xss研究到极致，然后在研究下一个课题xss通常结合csrf利用那么切忌不要因为xss会结合csrf使用就把研究重点放到csrf，要有重点的研究东西，一个月后可进行下一个课题研究。
  这是我写的一篇关于flash xss的总结希望能够帮到你们



[Flash XSS攻击总结**](//link.zhihu.com/?target=http%3A//www.secpulse.com/archives/44299.html%3Ffrom%3Dtimeline%26isappinstalled%3D1)

   2.整理笔记
​    这个我觉得最重要，这是一个好习惯 可以让我们重新把学习的技术做一个总结和巩固。在总结的过程形成自己对技术的理解与创新。

3.
   多看书和分析文章
比如我在学习代码审计的时候，最有效的方法
我是这样做的。
   代码审计在我们学习以前先去阅读相关的书籍。然后找两片分析0day的文章，照葫芦画瓢分析文章中一样版本的代码，这对你成长很有帮助，进步也非常快。
作者：怪力链接：https://www.zhihu.com/question/21606800/answer/77136580来源：知乎著作权归作者所有，转载请联系作者获得授权。