---
title: 建立这个博客的初衷与过程
time: 2015.11.30 21:47:00
layout: post
catalog: true
tags:
- Web
- 博客
excerpt: 我以前习惯在csdn上写博客，后来发现csdn的博客太丑，而且不是契合信息安全的主题。所以，我以jekyll在github上搭建了这个博客。
    



---

#建立这个博客的过程
##前言
我前面几年一直在csdn写博客。然而，越来越感觉csdn博客的界面比较丑。所以，打算这次用jekyll搭建一个个人博客。直至今日，博客的初始版本刚刚完成。
本博客主要采用github托管的方式，进行发布，是一个静态的网页，管理比较方便。在模版的采用方面，我借鉴了很多wenli博客的表现方式。在此，再次感谢wenli在我搭建博客中，提供的帮助。而由于最近又开始忙一些其它的项目。所以，这个博客的一些内容我仍然未完全迁移过来，仍发布在原网站上。所以可以说，这个网站仍未建设完毕，我打算寒假的时候，最终搭建完毕。也因此，如果你在浏览网页的同时，发现一些bug，请不要介怀。谢谢
##过程
###1.选择github pages的原因
很多人用 wordpress，那么我为什么要用 github pages 来搭建？

1. github pages有300M免费空间，资料自己管理，保存可靠；
2. 学着用 github，享受 github 的便利，上面有很多大牛，眼界会开阔很多；
3. 顺便看看 github 工作原理，最好的团队协作流程；
4. github 是趋势；
5. 就算 github 被墙了，还可以搬到国内的 gitcafe 中去。
6. 用github pages搭建博客很有big。

进一步了解：[pages](https://pages.github.com)


![image](http://cnfeat.qiniudn.com/bg2012082502.jpg)


###2.学习jekyll相关知识
1. jekyll主要就是将纯文本转化为静态网站和博客。
2. jekyll对博客很专注
3. 具体参考下列链接：
   [jekyll](http://jekyll.bootcss.com)

###3.学习Wenli Zhang的代码
1. 之所以选Wenli的代码，是因为一个偶然的契机－正好浏览到了她的网站，
   感觉她写的非常棒，而且也是用jekyll搭建的。所以，就发邮件联系了Wenli
   想她申请源码使用权。
2. 然后，我花了3天的时间，学习了她的[代码](https://github.com/Ovilia/blog)
   不得不说，wenli的代码写的非常漂亮。
3. 再参照她的代码，修改到了如今的版本。
4. 最后，在github pages中上线－[源码](https://github.com/momomoxiaoxi/momomoxiaoxi.github.io)

###4.购买域名
- 使用支付宝在 godaddy 购买域名：moxiaoxi.info.

- 本来是想买moxiaoxi.com的域名的，可惜被别人注册了＝ ＝

- 这里提醒大家一下，由于godaddy是国外的网站，所以购买会有延迟。如果你发现你支付了钱，缺仍未收到购买成功的信息，千万不要着急。10分钟左右就会成功。
- 具体流程参照：[购买过程](http://jingyan.baidu.com/album/6c67b1d6dac7e02787bb1e92.html)

###5.将独立域名与 GitHub Pages 的空间绑定
####DNS 设置
用 DNSpod，快，免费，稳定。
注册[DNSpod](https://www.dnspod.cn)，添加域名，如下图设置
![image](https://moxiaoxi.info/img/post/2015-11-01-how-i-made-this-site.png)
其中A的两条记录指向的ip地址是github Pages的提供的ip

- 192.30.252.153
- 192.30.252.154



如博客不能登录，有可能是 github 更改了空间服务的 ip 地址，记得及时到在GitHub Pages查看最新的ip即可

www 指定的记录是你在 github 注册的仓库。

####去 Godaddy 修改 DNS 地址
更改 godaddy 的 Nameservers 为 DNSpod 的 NameServers。
![image](http://cnfeat.qiniudn.com/16.png)

####6.测试博客
![image](https://moxiaoxi.info/img/post/2015-11-01-how-i-made-this-site-2.png)
测试成功，这样大体的博客框架就做好了。接下里就是写博文了：）
(由于这个博文是后期写的，所以可能有点简略。想详细了解的，可以联系我－｀－)




