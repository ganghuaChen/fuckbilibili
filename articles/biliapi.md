---
layout: page
permalink: "biliapi.html"
title:  "B站API"
---
                                                                
> 以下B站API全部由 [**Vespa314**](https://github.com/Vespa314/bilibili-api) 收集整理及开发。此 API 为人为整理，不能保证今后一直有效。

>此文的代码示例主要由 Python 实现。

{% include list.html %}

##直接爬取视频排行【已完成】:
* 获取URL: `http://www.bilibili.tv/list/[stow]-[zone]-[page]-[year1]-[month1]-[day1]~[year2]-[month2]-[day2].html`

* 返回：只关于视频部分的源码

###参数说明：

####Type:排序方式
* **收藏**：stow
* **评论数**：review
* **播放数**：hot
* **硬币数**：promote
* **用户评分**：comment
* **弹幕数**：damku
* *拼音*：pinyin-{x}，x可以是A~Z中的一个
* *投稿时间*：default(越新放在越前面)

>**注意：**上面排序方式中，**粗体字(前六个)**部分可以获取下文描述一切分区，但是*斜体(后两个)*只能获取二级以后的分区，也就是说**不可以**通过`拼音`和`投稿时间`来获取`综合排名`,`动画`,`音乐/舞蹈`,`游戏`,`科学技术`,`娱乐`,`影视`,`动画剧番`等分区。【可能是可以的，但是我没找到方法:-D】

####zone：分区
* **综合排名**：0
* **动画**：1
    * AMD·AMV：24
    * MMD·3D：25
    * 原创·配音：47
        * 原创：48
        * 中配：49
    * 二次元鬼畜：26
    * 综合：27
        * 手书：50
        * 咨询：51
        * 杂谈：52
        * 其他：53
* **音乐/舞蹈**：3
    * 音乐视频：28
        * OP/ED：54
        * 其他：55
    * Vocaloid相关：30
        * Vocaloid：56
        * UTAU相关：57
        * 中文曲：58
    * 翻唱：31
    * 舞蹈：20
    * 演奏：59
    * 三次元音乐：29
* **游戏**：4
    * 游戏视频：17
        * 预告·演示：61
        * 其他：63
    * 游戏攻略·解说：18
        * 单机游戏：64
        * 网络游戏：65
        * 家用·掌机：66
        * 其他：67
    * Mugen：19
    * 电子竞技：60
        * 赛事：68
        * 解说：69
        * 其它：70
* **科学技术**:36
    * 全球科技：39
        * 数码科技：95
        * 军事科技：96
        * 手机测评：97
        * 其它：98
    * 科普·人文：37
        * BBC纪录片：99
        * 探索频道：100
        * 国家地理：101
        * NHK：102
        * TED演讲：103
        * 名校公开课：104
        * 教程·演示：105
        * 其它：107
    * 野生技术协会：40
    * 趣味短片·其它：108
* **娱乐**：5
    * 生活娱乐：21
    * 三次元鬼畜：22
    * 动物圈：75
        * 喵星人：77
        * 汪星人：78
        * 其它：79
    * 美食：76
        * 美食视频：80
        * 制作教程：81
    * 综艺：71
* **影视**：11
    * 连载剧集：15
        * 国产：110
        * 日剧：111
        * 美剧：112
        * 其它：113
    * 完结剧集：34
        * 国产：87
        * 日剧：88
        * 美剧：89
        * 其它：90
    * 电影：23
        * 预告·花絮：82
        * 电影：83
    * 微电影：85
    * 特摄·布袋：86
        * 特摄：91
        * 布袋戏：92
* **动漫剧番**：13
    * 连载动画：33
    * 完结动画：32
    * 剧场·OVA：94

**注：page：页数，从1开始**

>如果只想查看原创，只需在后面加上`-original`即可，也就是URL=
`http://www.bilibili.tv/list/[stow]-[zone]-[page]-[year1]-[month1]-[day1]~[year2]-[month2]-[day2]-original.html`

#### API实现：
```python
def GetPopularVideo(begintime,endtime,sortType=TYPE_BOFANG,zone=0,page=1,original=0)
```

* 输入:
	* begintime：起始时间，三元数组[year1,month1,day1]
    * endtime：终止时间,三元数组[year2,month2,day2]
    * sortType：字符串，排序方式，参照TYPE_开头的常量
    * zone:整数，分区，参照api.md文档说明
    * page：整数，页数
* 返回：
    * 视频列表,包含AV号，标题，观看数，收藏数，弹幕数，投稿日期，封面，UP的id号和名字
* 备注：
    * 待添加：保证时间小于三个月
    * 待添加：TYPE_PINYIN模式后面要添加类似：TYPE_PINYIN-'A'
    
---

##视频Index
* 获取URL: `http://www.bilibili.tv/list/b-[firstlatter]-[zone]-[time]-[catalog]-[state]-[style]-[updatetime]-[sorttype]-[weekday]--[page].html`

* 返回：整个网页源代码


###参数说明：
#### zone:地区
* 不限：a
* 中国大陆：a1
* 日本：a2
* 美国：a3
* 英国：a4
* 加拿大：a5
* 中国香港：a6
* 中国台湾：a7
* 韩国：a8
* 法国：a9
* 德国：a15
* 其它：a16

#### time:上映时间
* 不限：留空即可【前后横杠要保留】
* 范围：1956-2014【有个别日期无法选择。。】

#### state:状态
* 不限：留空即可【前后横杠要保留】
* 连载中：0
* 完结：1

#### firstlatter:首字母
* 不限：留空即可【前后横杠要保留】
* A-Z：A-Z

#### weekday:星期
* 不限：留空即可【前后横杠要保留】
* 周日：0
* 周一到周六：分别取1~n

#### style:影片风格
* 不限：留空即可【前后横杠要保留】
* >1:萝莉  2:御姐 3:正太 4:后宫
5:百合  6:耽美  7:搞笑  8:恋爱
9:机战  10:热血  11:美少女  12:神魔
13:童话  14:教育  15:推理  16:惊悚
17:动作  18:励志  19:神话  20:治愈
21:致郁  22:男性向  23:女性向  24:校园
25:魔法  26:奇幻  27:魔幻  28:科幻
29:日常  30:泡面  31:冒险  32:竞技
33:运动  34:怪物  35:成人  36:真人
37:英雄  38:益智  39:生活  40:宠物
41:都市  42:假想科学  43:战斗
* 可以同时选择多种风格，用逗号隔开即可，如`1,2,4`,不过大部分不存在，要精确的话，可以选择先选择一种，然后根据返回源码即可得知当前风格加上什么风格可以有有效电影存在。

#### updatetime：更新时间
* 不限：留空即可【前后横杠要保留】
* 三天内：0
* 七天内：1
* 半月内：2
* 一月内：3

#### sorttype:排序方式
* 人气排序：a
* 更新排序：d
* 发布时间：n
* 播出日期：p

#### catalog：分类索引
* 全部：t
* 其它：t0
* TV：t1
* OVA&OAD：t2
* 剧场版：t3
* 连续剧：t5
* 电影：t6
* 微电影：t7
>很多没被分类的视频都无法在特定的索引分类下%>_<%

#### page：页数
必填，1~n

##按月份获取`动画`新番
* 获取URL: `http://www.bilibili.tv/index/bangumi/[year]-[month].json`

* 返回：json信息

###参数说明：
####输入：
* year:年份  四位数
* month:月份

返回：
* spid：spid
* weekday：番剧周信息
* title：标题
* cover:封面  【大】
* typeid:【不明】
* mcover:封面 【中】
* scover:封面 【小】

---

##B站API(无需认证或登录即可爬取的部分)：
**获取本周排行**
* URL：【返回json】
    * `http://api.bilibili.cn/index`
* 返回格式：
    * 第一层为type1，type3，type4，type5，type11，type13，type36，分别代表分区
    * 第二层为0~7表示第1到8名
    * 第三层为详细信息，包括：
        *  aid：AV号
        *  copyright：Original=原创，copy=转载
        *  typeid:更细致分区
        *  typename：类型名
        *  title：标题
        *  subtitle：副标题，很多视频没有
        *  play：播放数
        *  review：评论，少于浏览器看到的，包含回复
        *  video_review：弹幕数【估计包含被清过的】
        *  favorites：收藏
        *  mid：发布者账号
        *  author：发布者
        *  description：描述
        *  create：发布时间
        *  pic：封面
        *  credit：积分
        *  coins：硬币
        *  duration：时长
* 示例：
```
json['type1'][0]['title']
```

**读取作者推荐视频信息**
* URL：【返回json】
    * `http://api.bilibili.cn/author_recommend?aid=[id]`
* 输入：
    * aid：视频AV号 
* 返回格式：
    *  第一层：list
    *  第二层：0~n-1表示推荐的n部视频
    *  第三层：
        *  aid:av号
        *  title：标题
        *  cover：封面
        *  click：播放数
        *  review：评论，少于浏览器看到的，包含回复
        *  favorites：收藏
        *  video_review：弹幕数
* 示例
```
json['list']['0']['title']
```

**读取评论【已完成】**
* URL：【返回json】
    * `http://api.bilibili.cn/feedback`
* 输入：
    * aid：视频AV号
    * page：页码【选填】
    * pagesize：单页返回的记录条数，最大不超过300，默认为10。【选填】
    * ~~ver：API版本,最新是3【选填】~~
    * order：排序方式 默认按发布时间倒序 可选：good 按点赞人数排序 hot 按热门回复排序【选填】

> 官方说明ver选3，但是测试表明拿不回数据，ver应该是1
 
* 返回格式：
    *  第一层：
        *  totalResult：总评论数
        *  pages：页数
        *  0~n：评论条数[只有一页的]
    *  第二层(相对于评论条数)：
        *  mid:会员ID
        *  lv：楼层
        *  fbid：评论ID
        *  msg：评论信息
        *  ad_check：状态 (0: 正常 1: UP主隐藏 2: 管理员删除 3: 因举报删除)
        *  face：发布人头像
        *  rank：发布人显示标识
        *  nick：发布人暱称


#### API实现：
```python
# 获取视频单页评论
def GetComment(aid, page = None, pagesize = None, order = None)
# 获取视频全部评论
def GetAllComment(aid, order = None)
```

**读取专题信息**
* URL：【返回json】
    * `http://api.bilibili.tv/sp`
* 输入：
    * spid：专题编号【二选一】
    * title：专题名称【二选一】
* 返回格式：
    * spid：专题SPID
    * title：专题名
    * pubdate：发布日期 (UNIX Timestamp)
    * create_at：发布日期
    * lastupdate：最后更新日期 (UNIX Timestamp)
    * lastupdate_at：最后更新日期
    * alias：同义词
    * cover：封面
    * isbangumi：是否为新番 1=2次元新番 2=3次元新番
    * isbangumi_end：是否已经播放结束
    * bangumi_date：开播日期
    * description：专题简介
    * view：点击次数
    * favourite：专题收藏次数
    * attention：专题被关注次数
    * count：专题视频数量
* 示例
```python
http://api.bilibili.tv/sp?title=VOCALOID
```

**读取专题视频信息【已完成】**
* URL：【返回json】
    * ` http://api.bilibili.cn/spview`
* 输入：
    * spid：专题SPID
    * season_id：专题分季ID【选填】
    * bangumi：设置为1时只返回番剧类视频 设置为0时只返回普通视频 不设置则返回所有视频【选填】
> 经测试，设置为1返回剧番，不设置或者设置为0返回相关视频

* 返回格式：
    * 第一层：
        * count：视频数目
        * result：返回的记录总数目
        * list：包含视频信息
    * 第二层(相对于list)：
        * 编号 0~result-1表示第1~result个视频
    * 第三层(相对于编号) :
        * aid：AV号
        * cover：封面
        * title：标题
        * click：点击数
        * page：不明，全是0
        * from：视频源（如sohu，sina等）
        * cid：cid
        * episode：集数【注：不是所有专题视频都有此信息】

>说明：关于SPID的获取，暂时只知道`chrome`点击`F12`，然后查看`Network`中html的文件名编号
http://www.bilibili.tv/sppage/bangumi-[spid]-[page].html 也可以获得专题剧番的信息，有空补上说明
http://www.bilibili.tv/sppage/ad-recommend-[spid]-[page].html也可以获得相关专题信息。

#### API实现：
```python
def GetVideoOfZhuanti(spid,season_id=None,bangumi=None)
```

* 输入:
	* spid：见上
    * season_id：见上
    * bangumi：见上
* 返回：
    * 见上第三层
    


**读取用户信息【已完成】**

* URL：【返回json】
    * `http://api.bilibili.cn/userinfo`
* 输入：
    * user：昵称【二选一】
    * mid：用户ID【二选一】
* 返回格式：
    * mid：会员ID
    * name：暱名
    * approve：是否为认证帐号
    * spacename：空间名【注意：2015-01-29测试该项不返回】
    * sex：性别 (男/女/不明)
    * rank：帐号显示标识【实为用户等级, 和获取视频以及功能权限有关】
    * DisplayRank：【用户标识, 从 rank 衍生出, 影响实际显示的头像边框等】
    * face：小头像
    * attention：关注的好友人数
    * friend：好友数【不明白，反正比attention多1】
    * fans：粉丝人数
    * article：投稿数
    * place：所在地
    * description：认证用户为认证信息 普通用户为交友宣言
    * attentions：关注的好友列表
-* Rank 说明
-    * 32000: 站长 – 有权限获取所有视频信息 (包括未通过审核和审核中的视频)
-    * 31000: 职人
-    * 20000: 字幕君 – 有权限发送逆向弹幕
-    * 10000: 普通用户
-    * 参见 B 站 2012 年时因服务器配置错误泄露的部分 PHP 源码可更多了解权限机制: [https://gist.github.com/zacyu/8ed569008acff5ed05b5](https://gist.github.com/zacyu/8ed569008acff5ed05b5)

#### API实现
```python
GetUserInfoBymid(mid)
GetUserInfoByName(name)
```

**读取Up视频列表【已完成】**

* URL：【返回json】
    * `http://space.bilibili.com/ajax/member/getSubmitVideos`
* 输入：
    * mid：用户id
    * pagesize：单次拉去数目
    * page：页数
* 返回格式：
	* 第一层data
		* 第二层list
	    * aid：av号
	    * title：视频名
	    * copyright：是否原创
	    * typeid：tid
	    * typename：类别
	    * subtitle：子标题
	    * play：播放数
	    * review：评论数
	    * favorites：收藏数
	    * mid：Up主id
	    * author：Up主
	    * description：视频描述
	    * create：上传日期
	    * pic：封面URL
	    * credit：
	    * coins：硬币数
	    * duration：视频时长
	    * comment：弹幕数
		* count：视频总数目

#### API实现
```python
GetVideoOfUploader(mid,pagesize=20,page=1)
```

**读取Mylist列表信息**

* URL：【返回json】
    * `http://space.bilibili.com/ajax/fav/getboxlist`
* 输入：
    * mid：用户id
    * pagesize：单次拉去数目
    * page：页数
* 返回格式：
	* 第一层data
		* 第二层list
	    * fav_box：列表号码
	    * name：列表名
	    * max_count：容量
	    * count：数目
	    * atten_count：收藏人数
	    * state：【不明】
	    * ctime：【创建时间？！】
		* count：总数目

**读取订阅剧集信息**

* URL：【返回json】
    * `http://space.bilibili.com/ajax/bangumi/getlist`
* 输入：
    * mid：用户id
    * pagesize：单次拉去数目
    * page：页数
* 返回格式：
	* 第一层data
		* 第二层result
	    * season_id：【不知道什么鬼】
	    * share_url：专题所在页面
	    * title：标题
	    * is_finish：是否完结
	    * favorites：收藏人数
	    * newest_ep_index：最新剧集
	    * last_ep_index：【不知道什么鬼】
	    * total_count：总视频数目
	    * cover：封面
		* count：总数目
		* pages：页数


**读取投过硬币视频**

* URL：【返回json】
    * `http://space.bilibili.com/ajax/member/getcoinvideos`
* 输入：
    * mid：用户id
    * pagesize：单次拉去数目
    * page：页数
* 返回格式：
	* 第一层data
		* 第二层list
	    * aid：av号
	    * title：视频名
	    * copyright：是否原创
	    * typeid：tid
	    * typename：类别
	    * subtitle：子标题
	    * play：播放数
	    * review：评论数
	    * favorites：收藏数
	    * mid：Up主id
	    * author：Up主
	    * description：视频描述
	    * create：上传日期
	    * pic：封面URL
	    * credit：
	    * coins：硬币数
	    * duration：视频时长
	    * video_review：弹幕数
	    * comment：弹幕数
		* count：总数目
		* pages：页数

**读取订阅标签**

* URL：【返回json】
    * `http://space.bilibili.com/ajax/tags/getsublist`
* 输入：
    * mid：用户id
    * pagesize：单次拉去数目
    * page：页数
* 返回格式：
	* 第一层data
		* 第二层tags
	    * updated_ts：更新日期
	    * name：名称
	    * tag_id：标签编号
	    * tynotify：是否开启实时推送
		* count：总数目


**读取玩过的游戏**

* URL：【返回json】
    * `http://space.bilibili.com/ajax/game/getlastplay`
* 输入：
    * mid：用户id
    * pagesize：单次拉去数目
    * page：页数
* 返回格式：
	* 【找不到有数据的mid，没试出来】

**读取正在看的视频**

* URL：【返回json】
    * `http://space.bilibili.com/ajax/member/getViewing`
* 输入：
    * mid：用户id
* 返回格式：
	* aid:av号
	* title:视频名称

> 如果没在观看，返回『无数据』


**读取关注用户列表**

* URL：【返回json】
    * `http://space.bilibili.com/ajax/friend/GetAttentionList`
* 输入：
    * mid：用户id
    * pagesize：单次拉去数目
    * page：页数
* 返回格式：
	* 第一层data
		* 第二层list
	    * record_id：【不知道什么鬼】
	    * fid：被关注者mid
	    * addtime：关注事件
	    * uname：被关注者昵称
	    * face：头像
	    * attentioned：【不知道什么鬼，感觉不是全是1就是全是0】
		* pages：(当前pagesize下)总页数
		* results：总数目


**读取粉丝列表**

* URL：【返回json】
    * `http://space.bilibili.com/ajax/friend/GetFansList`
* 输入：
    * mid：用户id
    * pagesize：单次拉去数目
    * page：页数
* 返回格式：
	* 第一层data
		* 第二层list
	    * record_id：【不知道什么鬼】
	    * fid：粉丝mid
	    * addtime：关注事件
	    * uname：粉丝昵称
	    * face：头像
	    * attentioned：【不知道什么鬼，感觉不是全是1就是全是0】
		* pages：(当前pagesize下)总页数
		* results：总数目


---

##B站API(需认证)：
> 下方所有调用api方法均要加入`appkey=...`,如果是新注册的appkey的话还需要加入sign，具体算法是：
python：
```python
def GetSign(params, appkey, AppSecret=None):
    """
    获取新版API的签名，不然会返回-3错误
待添加：【重要！】
    需要做URL编码并保证字母都是大写，如 %2F
    """
    params['appkey']=appkey
    data = ""
    paras = params.keys()
    paras.sort()
    for para in paras:
        if data != "":
            data += "&"
        data += para + "=" + str(params[para])
    if AppSecret == None:
        return data
    m = hashlib.md5()
    m.update(data+AppSecret)
    return data+'&sign='+m.hexdigest()
```
js:
```Javascript
function get_sign(params, key)
 {
     var s_keys = [];
     for (var i in params)
     {
         s_keys.push(i);
     }
      s_keys.sort();
      var data = "";
      for (var i = 0; i < s_keys.length; i++)
     {
          // encodeURIComponent 返回的转义数字必须为大写( 如 %2F )
         data+=(data ? "&" : "")+s_keys[i]+"="+encodeURIComponent(params[s_keys[i]]);
     }
     return {
         "sign":hex_md5(data+key),
         "params":data
     };
 }
 ```


**读取视频信息**【已完成】
* URL：【返回json】
    * ` http://api.bilibili.cn/view`
* 输入：
    * id：AV号
    * page:页码
    * fav:是否读取会员收藏状态 (默认 0)【选填】

* 返回格式：
    * tid：【不明】
    * typename：【不明，总是返回`其他`】
    * instant_server：【不明】
    * spid:spid
    * src:【不明】
    * partname:【不明】
    * play：播放次数
    * review：评论数
    * video_review：弹幕数
    * favorites：收藏数
    * credit：评分数量
    * coins：推荐数量(硬币数)
    * title：标题
    * description：简介
    * tag：关键字
    * pic：封面图片URL地址
    * pages：返回记录的总页数
    * ~~from：视频来源~~
    * author：投搞人
    * mid：投搞人ID
    * cid：视频源及弹幕编号 弹幕地址 http://comment.bilibili.cn/<cid>.xml
    * offsite：Flash播放调用地址（如果沒有此项则此视频无法在站外播放）
    * create_at：视频发布日期
    * ~~favorited：当前帐号收藏状态~~

> 下载弹幕API：GetDanmukuContent(cid)，无需API验证

> **注意：**发现有部分视频必须登陆后才可以获得视频信息，这部分待完善！！！！

#### API实现：
```python
def GetVideoInfo(aid, appkey,page = 1, AppSecret=None, fav = None)
```


**获取新番信息**【已完成】
* URL：【返回json】
    * ` http://api.bilibili.cn/bangumi`
* 输入：
    * btype：番剧类型 2: 二次元新番 3: 三次元新番 默认：所有【选填】
    * weekday:周一:1 周二:2 ...周六:6 【选填】

> 官方API说周日的weekday取0，但经实验发现weekday=0返回全部信息，**目前不知如何获得周日新番列表**。。。

* 返回格式：
    * 第一层：
        * results：返回的记录总数目
        * count：返回的记录总数目
        * list：列表
    * 第二层：【相对于list】
        * 0~results-1：编号
    * 第三层：【相对于编号】
        * typeid:【不明】
        * lastupdate:最后更新时间  UNIX Timestamp
        * areaid:【不明】
        * bgmcount:番剧当前总集数
        * title:标题
        * lastupdate_at:最后更新时间
        * attention:【不明，跟视频热度有关】
        * cover:封面图片地址
        * priority:【不明】
        * area:地区  【如：日本】
        * weekday:番剧周信息  【这个时候周日又是0 TAT】
        * spid:spid
        * new:是否最近有更新
        * scover:封面图片地址  【为什么有两个。。】
        * mcover:封面图片地址  【为什么有三个。。】
        * click:浏览量
> 几个封面应该是不同大小的图片，但是不知为何返回来都是一样大的。。

**获取排行信息**【已完成】
* URL：【返回json】
    * `http://api.bilibili.cn/list`
* 输入：
    * tid: 分类编号 【new排序为必填 其他为可选】
    * exclude_tid: 排除分类编号 不可与tid同时使用【选填】
    * mid: 只显示该帐号投稿【选填】
    * begin: 起始搜索日期 (格式：YYYY-mm-dd)【选填】
    * end: 结束搜索日期 (格式：YYYY-mm-dd)【选填】
    * days:搜索最近天数 范围 1-90天 默认7天【选填】
    * page: 结果分页选择 默认为第1页【选填】
    * pagesize: 单页返回的记录条数，最大不超过100，默认为30【选填】
    * pinyin: 在使用拼音排序时必填选项 拼音首字母【选填】
    * click_detail: 返回详细点击信息【选填】
    * original: 只返回原创投稿【选填】
    * mission_id:返回活动投稿【选填】
    * exclude: 排除exclude中的id 可用,分隔多个【选填】
    * ver: 	 API版本 建议使用2【选填】
    * order: 排序方式【选填】
        * default	 按新投稿排序
        * new	 按新评论排序
        * review	 按评论数从高至低排序
        * hot	 按点击从高至低排序
        * damku	 按弹幕数从高至低排序
        * comment	 按推荐数从高至低排序
        * promote	 按宣传数排序（硬币）
        * pinyin	 按标题拼音排序

> **注：**
* 好像pagesize必填，不然出错。而且读取速度不能太快，返回速度也比较慢。
* click_detail好像只要填了非空，就可以获得`play_site`,`play_forward`,`play_mobile`三个数据

* 返回格式：
    * 第一层：
        * name:分类名称
        * results:返回的记录总数目
        * pages:返回的记录总页数
        * list:返回数据

    * 第二层：【相对于list】
        * 0~results-1：编号
    * 第三层：【相对于编号】
        * aid:视频编号
        * title:视频名称
        * copyright:是否原创
        * typeid:视频分类ID
        * typename:视频分类名称
        * subtitle:视频副标题
        * play:播放次数
        * review & comment:评论数【数值相同】
        * video_review:弹幕数
        * favorites:收藏数
        * author:视频作者
        * mid:视频作者ID
        * description:视频简介
        * create:视频创建日期
        * pic:封面图片地址
        * credit:评分数量
        * coins:硬币数量
        * duration:视频时长
        * play_site:网站播放
        * play_forward:【外链播放？】
        * play_mobile:app播放

错误代码：
* -601:起始日期格式错误
* -602:结束日期格式错误
* -603:选择的时间跨度过大
* -604:沒有输入拼音

#### API实现：
```python
def GetRank(appkey, tid, begin=None, end=None, page = None, pagesize=None, click_detail =None, order = None, AppSecret=None)
```

**搜索视频**【已完成】
* URL：【返回json】
    * `http://api.bilibili.cn/search`
* 输入：
    * keyword：关键词
    * order：排序方式  默认default，其余待测试【选填】
    * pagesize:返回条目多少【选填】
    * page：页码【选填】
* 返回：
	* aid：AV号
	* title：标题
	* typename：类型名
	* mid：上传者id
	* author：Up主姓名
	* arcurl：视频页面地址
	* description：视频简介
	* arcrank：视频状态与所需权限 (-6: 修复待审, -4: 撞车跳转, >0: 视频所需用户 rank)
	* pic：封面图片URL
	* play：播放数
	* video_review：弹幕数
	* favorites：收藏数
	* review：评论数
	* pubdate：上传时间
	* tag：标签
  
> 【**注意：**】经测试，此api的appkey必须使用新版的，即必须配合AppSecret使用，否则会返回{“code”:-3,”message”:”API sign invalid”}

#### API实现：
```python
def biliVideoSearch(appkey, AppSecret, keyword, order = ‘default’, pagesize = 20, page = 1)
```

**搜索专题**【已完成】
* URL：【返回json】
    * `http://api.bilibili.cn/search`
* 输入：
    * keyword：关键词
* 返回：
	* spid：spid
	* title：标题
	* mid：Up的id
	* author：Up
	* pic：封面url
	* thumb：小封面url
	* ischeck：【不明】
	* tag：tag
	* description：说明
	* pubdate：【不明】
	* postdate：【不明】
	* lastupdate：最后更新时间
	* click：点击
	* favourite：收藏
	* attention：关注
	* count：数目
	* bgmcount：番剧数目
	* spcount：【不明】
	* season_id：专题分季id
	* is_bangumi：是否番剧
	* arcurl：专题页面URL
  
> 【**注意：**】经测试，此api的appkey必须使用新版的，即必须配合AppSecret使用，否则会返回{“code”:-3,”message”:”API sign invalid”}

#### API实现：
```python
biliZhuantiSearch(appkey, AppSecret, keyword)
```

---
## 辅助API

**获取视频下载URL**【已完成】

code：
GetBilibiliUrl(url,appkey,AppSecret=None)

* 输入：
	* url：视频地址，比如：
		* http://www.bilibili.com/video/av510515/index_3.html
		* http://www.bilibili.com/video/av1687261/
	* appkey：
	* AppSecret：密钥什么的。。。
* 返回
	* url列表

> 返回URL地址经常变化，而且有时返回的是若干个6分钟短视频的url，需要通过ffmpeg等工具连接起来，但是偶尔再等一下返回的又变成一个URL了。。。

**获取弹幕信息**【已完成】

code：
ParseDanmuku(cid)

* 输入：
	* cid：视频cid
* 返回
	* 弹幕信息：
		* danmu.t_video:弹幕时间（基于视频）
		* danmu.t_stamp:弹幕发表时间
		* danmu.mid_crc:用户mid经过crc32计算后的16进制结果值为:hex(binascii.crc32(mid))
		* danmu.danmu_type:弹幕类型
			* 1：滚动弹幕
			* 4：底部弹幕
			* 5：顶部弹幕
		* danmu.content：弹幕内容
		* danmu.danmu_color：弹幕颜色

***

点开导航栏中的 *Vespa314* 页面，你会发现更多有趣的东西~
