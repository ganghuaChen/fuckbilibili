---
layout: page
permalink: "srt2bilibili.html"
title:  "批量发弹幕器"
---

小心作死。

## 事前唠个叨

为什么这么说？据可(kai)靠(fa)人(zhe)士(zi)说(ji)，这个“神器”几乎是不管把发弹幕间隔设多长都会导致封号。

所以要小心地作死。

### 源码如下：

{% highlight python %}
#!/usr/bin/env python
#coding:utf-8
# Author:  Beining --<ACICFG>
# Purpose: A batch poster of srt file to danmaku on Bilibili.
# Created: 11/23/2014
# srt2Bilibili is licensed under GNUv2 license
'''
srt2Bilibili 0.02.2 alpha
Beining@ACICFG
cnbeining[at]gmail.com
http://www.cnbeining.com
https://github.com/cnbeining/srt2bilibili
GNUv2 license
'''

import sys
if sys.version_info < (3, 0):
    sys.stderr.write('ERROR: Python 3.0 or newer version is required.\n')
    sys.exit(1)
try:
    import requests
except:
    sys.stderr.write('ERROR: Requests is required. Please check https://github.com/cnbeining/srt2bilibili#usage .\n')
    sys.exit(1)
try:
    import pysrt
except:
    sys.stderr.write('ERROR: Pysrt is required. Please check https://github.com/cnbeining/srt2bilibili#usage .\n')
    sys.exit(1)

import os
import random
import urllib
import logging
import hashlib
import time as time_old
import getopt
from xml.dom.minidom import parse, parseString
import xml.dom.minidom
from random import randint

global APPKEY, SECRETKEY, VER, rnd, cid

APPKEY = '85eb6835b0a1034e'
SECRETKEY = '2ad42749773c441109bdc0191257a664'
VER = '0.02.2 alpha'

#----------------------------------------------------------------------
def calc_sign(string):
    """str/any->str
    return MD5.
    From: Biligrab, https://github.com/cnbeining/Biligrab
    MIT License"""
    return str(hashlib.md5(str(string).encode('utf-8')).hexdigest())

#----------------------------------------------------------------------
def find_cid_api(vid, p):
    """find cid and print video detail
    str,int?,str->str
    TODO: Use json.
    From: Biligrab, https://github.com/cnbeining/Biligrab
    MIT License"""
    cid = 0
    if str(p) is '0' or str(p) is '1':
        str2Hash = 'appkey={APPKEY}&id={vid}&type=xml{SECRETKEY}'.format(APPKEY = APPKEY, vid = vid, SECRETKEY = SECRETKEY)
        biliurl = 'https://api.bilibili.com/view?appkey={APPKEY}&id={vid}&type=xml&sign={sign}'.format(APPKEY = APPKEY, vid = vid, SECRETKEY = SECRETKEY, sign = calc_sign(str2Hash))
    else:
        str2Hash = 'appkey={APPKEY}&id={vid}&page={p}&type=xml{SECRETKEY}'.format(APPKEY = APPKEY, vid = vid, p = p, SECRETKEY = SECRETKEY)
        biliurl = 'https://api.bilibili.com/view?appkey={APPKEY}&id={vid}&page={p}&type=xml&sign={sign}'.format(APPKEY = APPKEY, vid = vid, SECRETKEY = SECRETKEY, p = p, sign = calc_sign(str2Hash))
    logging.debug(biliurl)
    logging.info('Fetching webpage...')
    try:
        request = urllib.request.Request(biliurl, headers=BILIGRAB_HEADER)
        response = urllib.request.urlopen(request)
        data = response.read()
        dom = parseString(data)
        for node in dom.getElementsByTagName('cid'):
            if node.parentNode.tagName == "info":
                cid = node.toxml()[5:-6]
                logging.info('cid is ' + cid)
                break
        return cid
    except:  # If API failed
        logging.warning('Cannot connect to API server! \nIf you think this is wrong, please open an issue at \nhttps://github.com/cnbeining/Biligrab/issues with *ALL* the screen output, \nas well as your IP address and basic system info.')
        return 0

#----------------------------------------------------------------------
def convert_cookie(cookie_raw):
    """str->dict
    'DedeUserID=358422; DedeUserID__ckMd5=; SESSDATA=72e0ee97%%2C6b47a180'
    cookie = {'DedeUserID': 358422, 'DedeUserID__ckMd5': '', 'SESSDATA': '72e0ee97%%2C6b47a180'}"""
    cookie = {}
    logging.debug('Raw Cookie: ' + cookie_raw)
    try:
        for i in [i.strip() for i in cookie_raw.split(';')]:
            cookie[i.split('=')[0]] = i.split('=')[1]
    except IndexError:
        #if someone put a ; at the EOF
        pass
    return cookie

#----------------------------------------------------------------------
def getdate():
    """None->str
    2014-11-23 10:39:46"""
    return time_old.strftime("%Y-%m-%d %X", time_old.localtime())

#----------------------------------------------------------------------
def post_one(message, rnd, cid, cookie, fontsize = 25, mode = 1, color = 16777215, playTime = 0, pool = 0, fake_ip = False):
    """
    PARS NOT THE PERFECT SAME AS A PAYLOAD!"""
    headers = {'Origin': 'http://static.hdslb.com', 'X-Requested-With': 'ShockwaveFlash/15.0.0.223', 'Referer': 'http://static.hdslb.com/play.swf', 'User-Agent': BILIGRAB_UA, 'Host': 'interface.bilibili.com', 'Content-Type': 'application/x-www-form-urlencoded', 'Cookie': cookie}
    if fake_ip:
        FAKE_IP = ".".join(str(randint(1, 255)) for i in range(4))
        headers.update({'X-Forwarded-For' : FAKE_IP, 'Client-IP' : FAKE_IP})
    #print(headers)
    url = 'http://interface.bilibili.com/dmpost'
    try:
        date = getdate()
        payload = {'fontsize': int(fontsize), 'message': str(message), 'mode': int(mode), 'pool': pool, 'color': int(color), 'date': str(date), 'rnd': int(rnd), 'playTime': playTime, 'cid': int(cid)}
        encoded_args = urllib.parse.urlencode(payload)
        r = requests.post(url, data = encoded_args, headers = headers)
        #print(r.text)
        if int(r.text) <= 0:
            logging.warning('Line failed:')
            logging.warning('Message:' + str(message))
            logging.warning('ERROR Code: ' + str(r.text))
        else:
            print(message)
        #logging.info(message)
    except Exception as e:
        print('ERROR: Line failed: %s' % e)
        print('Payload:' + str(payload))
        pass

#----------------------------------------------------------------------
def timestamp2sec(timestamp):
    """SubRipTime->float
    SubRipTime(0, 0, 0, 0)"""
    return (int(timestamp.seconds) + 60 * int(timestamp.minutes) + 3600 * int(timestamp.hours) + float(int(timestamp.hours) / 1000))

#----------------------------------------------------------------------
def read_cookie(cookiepath):
    """str->list
    Original target: set the cookie
    Target now: Set the global header
    From: Biligrab, https://github.com/cnbeining/Biligrab
    MIT License"""
    global BILIGRAB_HEADER
    try:
        cookies_file = open(cookiepath, 'r')
        cookies = cookies_file.readlines()
        cookies_file.close()
        # print(cookies)
        return cookies
    except:
        logging.warning('Cannot read cookie!')
        return ['']

#----------------------------------------------------------------------
def main(srt, fontsize, mode, color, cookie, aid, p = 1, cool = 0.1, pool = 0, fake_ip = False):
    """str,int,int,int,str,int,int,int,int->None"""
    rnd = int(random.random() * 1000000000)
    cid = int(find_cid_api(aid, p))
    subs = pysrt.open(srt)
    for sub in subs:
        #lasttime = timestamp2sec(sub.stop) - timestamp2sec(sub.start)
        # For future use
        playtime = timestamp2sec(sub.start)
        message = sub.text
        if '\n' in message:
            for line in message.split('\n'):
                post_one(line, rnd, cid, cookie, fontsize, mode, color, playtime, pool,fake_ip = fake_ip)
                time_old.sleep(float(cool))
        else:
            post_one(message, rnd, cid, cookie, fontsize, mode, color, playtime, pool,fake_ip = fake_ip)
            time_old.sleep(float(cool))
    print('INFO: DONE!')


#----------------------------------------------------------------------
def usage():
    """"""
    print('''
    srt2Bilibili
    
    https://github.com/cnbeining/srt2bilibili
    http://www.cnbeining.com/
    
    Beining@ACICFG
    
    WARNING: THIS PROGRAMME CAN BE DANGEROUS IF MISUSED,
    AND CAN LEAD TO UNWANTED CONSEQUNCES,
    INCLUDING (BUT NOT LIMITED TO) TEMPORARY OR PERMANENT BAN OF ACCOUNT AND/OR
    IP ADDRESS, DANMAKU POOL OVERSIZE, RUIN OF NORMAL DANMAKU.
    
    ONLY USE WHEN YOU KNOW WHAT YOU ARE DOING.
    
    This program is provided **as is**, with absolutely no warranty.
    
    
    Usage:
    
    python3 srt2bilibili.py (-h) (-a 12345678) [-p 1] [-c ./bilicookies] (-s 1.srt) [-f 18] [-m 0] [-o 16711680] [-w 0.1] [-l 0] (-i)
    
    -h: Default: None
        Print this usage file.
        
    -a: Default: None
        The av number.
        
    -p: Default: 1
        The part number.
        
    -c Default: ./bilicookies
        The path of cookies.
        Should looks like:
        
        DedeUserID=123456;DedeUserID__ckMd5=****************;SESSDATA=*******************
            
    -s Default: None
        The srt file you want to post.
        srt2bilibili will post multi danmakues for multi-line subtitle,
        since there's a ban on the use of \n.
        
    -f Default: 18
        The size of danmaku.
        
    -m Default: 4
        The mode of danmaku.
        1: Normal
        4: Lower Bound  *Suggested
        5: Upper Bound
        6: Reverse
        7: Special
        9: Advanced
        
    -o Default: 16711680
        The colour of danmaku, in integer.
        Default is red.
        
    -w Default: 0.1
       The cool time (time to wait between posting danmakues)
       Do not set it too small, which would lead to ban or failure.
       
    -l Default: 0
        The Danmaku Pool to use.
        0: Normal
        1: Subtitle
        2: Special
        If you own the video, please set it to 1 to prevent potential lost of danmaku.
        
    -i Default: False
        Use a fake IP address for every comment.
    
    More info avalable at http://docs.bilibili.cn/wiki/API.comment  .
    ''')

#----------------------------------------------------------------------
if __name__=='__main__':
    argv_list = []
    argv_list = sys.argv[1:]
    aid, part, cookiepath, srt, fontsize, mode, color, cooltime, playtime, pool = 0, 1, './bilicookies', '', 18, 4, 16711680, 0.1, 0, 0
    try:
        opts, args = getopt.getopt(argv_list, "ha:p:c:s:f:m:o:w:l:i",
                                   ['help', "av", 'part', 'cookie', 'srt', 'fontsize', 'mode', 'color', 'cooltime', 'pool', 'fake-ip'])
    except getopt.GetoptError:
        usage()
        exit()
    for o, a in opts:
        if o in ('-h', '--help'):
            usage()
            exit()
        if o in ('-a', '--av'):
            aid = a
            try:
                argv_list.remove('-a')
            except:
                break
        if o in ('-p', '--part'):
            part = a
            try:
                argv_list.remove('-p')
            except:
                part = 1
                break
        if o in ('-c', '--cookie'):
            cookiepath = a
            try:
                argv_list.remove('-c')
            except:
                print('INFO: No cookie path set, use default: ./bilicookies')
                cookiepath = './bilicookies'
                break
        if o in ('-s', '--srt'):
            srt = a
            try:
                argv_list.remove('-s')
            except:
                break
        if o in ('-f', '--fontsize'):
            fontsize = a
            try:
                argv_list.remove('-f')
            except:
                break
        if o in ('-m', '--mode'):
            mode = a
            try:
                argv_list.remove('-m')
            except:
                mode = 1
                break
        if o in ('-o', '--color'):
            color = a
            try:
                argv_list.remove('-o')
            except:
                color = 16711680
                break
        if o in ('-w', '--cooltime'):
            cooltime = a
            try:
                argv_list.remove('-w')
            except:
                cooltime = 0.1
                break
        if o in ('-l', '--pool'):
            pool = a
            try:
                argv_list.remove('-l')
            except:
                pool = 0
                break
        if o in ('-i', '--fake-ip'):
            fake_ip = True
    if aid == 0:
        logging.fatal('No aid!')
        exit()
    if srt == '':
        logging.fatal('No srt!')
        exit()
    if len(cookiepath) == 0:
        cookiepath = './bilicookies'
    cookies = read_cookie(cookiepath)
    logging.debug('Cookies: ' + cookiepath)
    BILIGRAB_UA = 'srt2Bilibili / ' + str(VER) + ' (cnbeining@gmail.com)'
    BILIGRAB_HEADER = {'User-Agent': BILIGRAB_UA, 'Cache-Control': 'no-cache', 'Pragma': 'no-cache', 'Cookie': cookies[0]}
    logging.debug(cookies[0])
    main(srt, fontsize, mode, color, cookies[0], aid, part, cooltime, pool, fake_ip = fake_ip)
{% endhighlight %}

大家可以在[这里](/script/srt2bilibili.py)或者 [Github](https://github.com/cnbeining/srt2bilibili) 上找到源码。

##用法

{% highlight bash %}
~ $ python3 srt2bilibili.py (-h) (-a 12345678) [-p 1] [-c ./bilicookies] (-s 1.srt) [-f 18] [-m 0] [-o 16711680] [-w 0.1] [-l 0] (-i)

-h: 默认：无  
    得到帮助

-a: 默认：无  
    av号

-p: 默认：1  
    分p号

-c 默认：./bilicookies  
    cookies文件  
    看起来应该是这样的：  

    DedeUserID=123456;DedeUserID__ckMd5=****************;SESSDATA=*******************

-s 默认：无  
    你想发的srt文件  
    srt2bilibili会将多行字幕转换成多行弹幕发出去  
    因为不让用\N

-f 默认：18  
    弹幕大小

-m 默认：4  
    弹幕类型  
    1：普通  
    4：底端  *建议  
    5：顶端  
    6：逆行  
    7：特殊  
    9：高级

-o 默认：16711680  
    弹幕的颜色，以整数形式  
    默认是红色

-w 默认：0.1  
   冷却时间（发弹幕的间隔时间）  
   别设太小，不过用这个几乎就是会被禁

-l 默认：0  
    使用的弹幕池  
    0：普通  
    1：字幕  
    2：特殊  
    如果你是up主，请设成1以避免弹幕的消失

-i 默认：没有  
    用假ip发弹幕
{% endhighlight %}

更多信息[http://docs.bilibili.cn/wiki/API.comment](http://docs.bilibili.cn/wiki/API.comment){:target="_blank"}

##需求：

* Python 3.x
* pysrt `pip3 install pysrt`
* requests `pip3 install requests`
