---
layout: page
permalink: "bililivesilver.html"
title:  "自动领直播间银瓜子"
---
<style type="text/css">
  input[type=text]{border:2px solid #ccc;color:#000;font-size:1em;padding:10px 15px 10px 15px;}
  #container{text-align: center;}
  #cookie_input{width: 100%;height: 100%}
</style>

## 事前唠个叨

[Beining](https://www.cnbeining.com/){:target="_blank"}又无聊写了个自动领B站直播间银瓜子的程序，我改了改，放了上来~

最下面有惊喜哦~

### Python示例源码如下：
勿吐槽两种Python写法交集……

{% highlight python %}
#!/usr/bin/env python
#coding:utf-8
# Author:  Beining --<i#cnbeining.com>
# Co-op: SuperFashi
# Purpose: Auto grab silver of Bilibili
# Created: 10/22/2015
# https://www.cnbeining.com/
# https://github.com/cnbeining

import sys
import os
import requests
import getopt
from json import loads
import datetime
import time
import re
import logging
import traceback
try:
    from baiduocr import BaiduOcr
except ImportError:
    print('你需要BaiduOcr组件。')
    print('请访问https://github.com/Linusp/baidu_ocr')
    exit()


# Dual support
try:
    input = raw_input
except NameError:
    pass

# LATER
#BAIDU_KEY =

#----------------------------------------------------------------------
def logging_level_reader(LOG_LEVEL):
    """str->int
    Logging level."""
    return {
        'INFO': logging.INFO,
        'DEBUG': logging.DEBUG
    }.get(LOG_LEVEL)

#----------------------------------------------------------------------
def generate_16_integer():
    """None->str"""
    from random import randint
    return str(randint(1000000000000000, 9999999999999999))

#----------------------------------------------------------------------
def safe_to_eval(string_this):
    """"""
    pattern = re.compile(r'^[\d\+\-\s]+$')
    match = pattern.match(string_this)
    if match:
        return True
    else:
        return False

#----------------------------------------------------------------------
def get_new_task_time_and_award(headers):
    """dict->tuple of int
    time_in_minutes, silver"""
    random_r = generate_16_integer()
    url = 'http://live.bilibili.com/FreeSilver/getCurrentTask?r=0.{random_r}'.format(random_r = random_r)
    response = requests.get(url, headers=headers)
    a = loads(response.content.decode('utf-8'))
    logging.debug(a)
    if a['code'] == 0:
        return (a['data']['minute'], a['data']['silver'])

#----------------------------------------------------------------------
def get_captcha_from_live(headers):
    """dict,str->str
    get the captcha link"""
    random_t = generate_16_integer()  #save for later
    url = 'http://live.bilibili.com/FreeSilver/getCaptcha?t=0.{random_t}'.format(random_t = random_t)
    response = requests.get(url, stream=True, headers=headers)
    filename = random_t + ".jpg"
    with open(filename, "wb") as f:
        f.write(response.content)
    result = os.path.abspath(filename)
    logging.debug(result)
    return result

#----------------------------------------------------------------------
def image_link_ocr(image_link):
    """link can be local file"""

    API_KEY = 'c1ff362dc90585fed08e80460496eabd'
    client = BaiduOcr(API_KEY, 'test')  # 使用个人免费版 API，企业版替换为 'online'

    res = client.recog(image_link, service='Recognize', lang='CHN_ENG')
    os.remove(image_link)
    logging.debug(res)

    return res['retData'][0]['word']

#----------------------------------------------------------------------
def send_heartbeat(headers):
    """"""
    random_t = generate_16_integer()
    url = 'http://live.bilibili.com/freeSilver/heart?r=0.{random_t}'.format(random_t = random_t)
    response = requests.get(url, headers=headers)
    a = loads(response.content.decode('utf-8'))
    if a['code'] != 0:
        return False
    elif response.status_code != 200:
        print('错误：心跳发送失败！') # Probably never see this
        return False
    else:
        return True

#----------------------------------------------------------------------
def get_award(headers, captcha):
    """dict, str->int/str"""
    url = 'http://live.bilibili.com/freeSilver/getAward?r=0.{random_t}&captcha={captcha}'.format(random_t = generate_16_integer(), captcha = captcha)
    response = requests.get(url, headers=headers)
    a = loads(response.content.decode('utf-8'))
    if response.status_code != 200 or a['code'] != 0:
        print(a['msg'])
        return [int(a['code']), 0]
    else:
        return [int(a['data']['awardSilver']), int(a['data']['silver'])]

#----------------------------------------------------------------------
def award_requests(headers):
    url = 'http://live.bilibili.com/freeSilver/getSurplus?r=0.{random_t}'.format(random_t = generate_16_integer())
    response = requests.get(url, headers=headers)
    a = loads(response.content.decode('utf-8'))
    if response.status_code != 200 or a['code'] != 0:
        return False
    else:
        return True

#----------------------------------------------------------------------
def read_cookie(cookiepath):
    """str->list
    Original target: set the cookie
    Target now: Set the global header"""
    print(cookiepath)
    try:
        cookies_file = open(cookiepath, 'r')
        cookies = cookies_file.readlines()
        cookies_file.close()
        return cookies
    except Exception:
        return ['']

#----------------------------------------------------------------------
def captcha_wrapper(headers):
    """"""
    captcha_link = get_captcha_from_live(headers)
    captcha_text = image_link_ocr(captcha_link).encode('utf-8')
    answer = ''
    if safe_to_eval(captcha_text):
        try:
            answer = eval(captcha_text)  #+ -
        except NameError:
            answer = ''
    return answer

#----------------------------------------------------------------------
def usage():
    """"""
    print("""Auto-grab

    -h: 帮助:
    这个。

    -c: Cookies:
    默认: ./bilicookies
    Cookie的位置
    
    -l: 除错log
    默认: INFO
    INFO/DEBUG
    """)

#----------------------------------------------------------------------
def main(headers = {}):
    """"""
    try:
        time_in_minutes, silver = get_new_task_time_and_award(headers)
    except TypeError:
        print('你今天的免费银瓜子已经领完了，明天再来吧~')
        exit()
    print('预计下一次领取需要{time_in_minutes}分钟，可以领取{silver}个银瓜子'.format(time_in_minutes = time_in_minutes, silver = silver))
    now = datetime.datetime.now()
    picktime = now + datetime.timedelta(minutes = time_in_minutes) + datetime.timedelta(seconds = 10)
    while (picktime - datetime.datetime.now()).seconds / 60 > 0 and ((picktime - datetime.datetime.now()).seconds / 60) <= 10:
        if not send_heartbeat(headers):
            print('还剩下'+str((picktime - datetime.datetime.now()).seconds / 60)+'分钟……')
            time.sleep(60)
    while not award_requests(headers):
        time.sleep(10)
    answer = 0
    award, nowsilver = [0, 0]
    print('开始领取！')
    for i in range(1, 11):
        answer = captcha_wrapper(headers)
        count = 1
        while answer == '':
            print('验证码识别错误，重试第'+str(count)+'次')
            answer = captcha_wrapper(headers)
            count += 1
        award, nowsilver = get_award(headers, answer)
        if award > 0:
            break
        else:
            print('错误，重试第{i}次'.format(i = i))
            time.sleep(5)
    print('成功！得到'+str(award)+'个银瓜子，你现在有'+str(nowsilver)+'个银瓜子')
    return award

if __name__=='__main__':
    argv_list = []
    argv_list = sys.argv[1:]
    cookiepath,LOG_LEVEL = '', ''
    try:
        opts, args = getopt.getopt(argv_list, "hc:l:",
                                   ['help', "cookie=", "log="])
    except getopt.GetoptError:
        usage()
        exit()
    for o, a in opts:
        if o in ('-h', '--help'):
            usage()
            exit()
        if o in ('-c', '--cookie'):
            cookiepath = a
            # print('aasd')
        if o in ('-l', '--log'):
            try:
                LOG_LEVEL = str(a)
            except Exception:
                LOG_LEVEL = 'INFO'
    logging.basicConfig(level = logging_level_reader(LOG_LEVEL))
    if cookiepath == '':
        cookiepath = './bilicookies'
    if not os.path.exists(cookiepath):
        print('Cookie文件未找到！')
        print('请将你的Cookie信息放到\"'+cookiepath+'\"里')
        exit()
    cookies = read_cookie(cookiepath)[0]
    if cookies == '':
        print('不能读取Cookie文件，请检查')
        exit()
    headers = {
        'accept-encoding': 'gzip, deflate, sdch',
        'user-agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_11_1) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/47.0.2526.16 Safari/537.36',
        'authority': 'live.bilibili.com',
        'cookie': cookies,
    }
    while 1:
        try:
            main(headers)
        except KeyboardInterrupt:
            exit()
        except Exception as e:
            print('错误！ {e}'.format(e = e))
            traceback.print_exc()
{% endhighlight %}

##用法

{% highlight bash %}
~ $ python autograb.py

-h: 帮助:

-c: Cookies:
默认: ./bilicookies
Cookie的位置
    
-l: 除错log
默认: INFO
INFO/DEBUG
{% endhighlight %}

大家可以在[这里](/script/autograb.py){:target="_blank"}或者去 [Github](https://github.com/fuckbilibili/Bilibili-Silver-Grab){:target="_blank"} 上找到源码。

***

什么？上面的都看不懂？？直接给你免费用吧，真是的（＞д＜）

<div id="container">
  <form method="post" id="downform" name="downform" action="//dynamic.fuckbilibili.com/script/get.php">
  <p>将你的b站Cookie直接丢到这里即可~</p>
    <input type="text" name="Cookie" id="cookie_input" placeholder="LIVE_BUVID=xxx; LIVE_BUVID__ckMd5=xxx; DedeUserID=xxx; DedeUserID__ckMd5=xxx; SESSDATA=xxx; LIVE_LOGIN_DATA=xxx; LIVE_LOGIN_DATA__ckMd5=xxx; DedeID=xxx" />
    <input type="submit" style="display: none;" />
  </form>
</div>

    领取分3次循环，每次循环为3小部分：
    3分钟等待，拿30个；6分钟等待，拿80个；10分钟等待，拿190个。
    总共需要1个小时左右，总共可以每天拿900个。
    数据库每天凌晨清零，觉得好用的话第二天再来提交哦~
    ※获取Cookie之前建议重新登录一次※
    ※提交后两个小时之内最好不要碰B站帐号※
    ※随时可以将你提交过的Cookie再次提交来查询目前状态※
    ！使用前请阅读下方的使用须知！

### 怎么获取Cookie？
打开b站[直播主页](http://live.bilibili.com){:target="_blank"}，按F12打开浏览器调试工具，换到console（控制台）一栏，输入以下命令：

```document.cookie```

把获取到的Cookie直接丢上来就行了~（不带最外引号）

### 使用须知

提供你的Cookie相当于可以直接登录你的账号，包括但不限于投递稿件，删除稿件，发送弹幕，删除弹幕，发送私信，阅读私信，删除私信，修改空间，修改邮箱，修改手机，修改QQ连接，修改签名，投稿投诉，弹幕投诉，进行直播，停止直播，尝试反推密码等。

你的Cookie将会加密传输，明文存储至我们的数据库，你的Cookie将每天凌晨被删除。我们不会使用你的Cookie进行除领取银瓜子之外的任何操作，并且你的Cookie泄露的可能性微乎其微。所有的操作源码以GPLv3协议开源在Github上。

但决定权还是在你手里，若你提交你的Cookie，则默认阅读过并同意以上须知。

***

不想提交cookie，又不会用python，懒死你得了！

## 使用js版吧！

    只要将下面图片拖动到浏览器的书签栏
    再在任意直播房间页面点击书签即可
    注：请不要在多个页面同时使用（并不能加快瓜子领取）

<a style="background: url(/public/img/1.png) no-repeat 0 0;
    cursor: move;
    display: block;
    height: 56px;
    width: 290px;
    margin: auto;" href="javascript:void (function(a,d){d=document.createElement('script');d.src=a;document.body.appendChild(d)})('http://pa001024.github.io/BiliSilverCatcher/bsc.js')" class="bookmark-drop-btn" alt="拖动我到书签栏">
	<span style="visibility: hidden;">❤领瓜子</span>
</a>
