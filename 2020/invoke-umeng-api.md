---
2020-05-27 16:58
---

# 如何调用友盟API进行消息推送以及遇到的坑

**因为不同业务的需求不同，这里只是简单DEMO**

## 代码

```python
import hashlib
import json
import time


# 计算md5的函数
def md5(s):
    return hashlib.new("md5", s.encode("utf8")).hexdigest()


# 消息描述
description = '老黄历'
# 要推送的消息
title = '今日运势'
message = '宜上分；忌打码'
# 过期时间
expireTime = '2020-05-28 14:11:54'
appkey = '5ece088f0cafb2b1b200015f'
app_master_secret = 'a8lkbfq8sy1v24tmszy4iefmsgdrrbk1'

# >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
timestamp = int(round(time.time() * 1000))
method = 'POST'
url = 'https://msgapi.umeng.com/api/send'
params = {
    "policy": {
        "expire_time": expireTime
    },
    "description": description,
    "production_mode": True,
    "appkey": appkey,
    "payload": {
        "body": {
            "title": title,
            "ticker": title,
            "text": message,
            "after_open": "go_app",
            "play_vibrate": "false",
            "play_lights": "false",
            "play_sound": "true"
        },
        "display_type": "notification"
    },
    "type": "broadcast",
    "timestamp": timestamp
}

post_body = json.dumps(params)
# 计算签名
sign = md5('%s%s%s%s' % (method, url, post_body, app_master_secret))
print()
# 输出接口地址
print(url + '?sign=' + sign)
print()
# 输出请求体
print(post_body)
# <<<<<<<<<<<<<<<<<<<<<<<<<<<

```
运行后输出

![20200527165235130](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2020/image/20200527165235130.png)
为什么要把输出的图片贴出来呢？因为这里有下面要说的一个坑，如果只是复制下面的输出就遇不到，但是如果是复制控制台的输出很可能就会遇到。

```
https://msgapi.umeng.com/api/send?sign=8ca87b046ce71c04f329bd17b05ad287

{"policy": {"expire_time": "2020-05-28 14:11:54"}, "description": "\u8001\u9ec4\u5386", "production_mode": true, "appkey": "5ece088f0cafb2b1b200015f", "payload": {"body": {"title": "\u4eca\u65e5\u8fd0\u52bf", "ticker": "\u4eca\u65e5\u8fd0\u52bf", "text": "\u5b9c\u4e0a\u5206\uff1b\u5fcc\u6253\u7801", "after_open": "go_app", "play_vibrate": "false", "play_lights": "false", "play_sound": "true"}, "display_type": "notification"}, "type": "broadcast", "timestamp": 1590569405625}
```
然后复制`URL`和**参数**去模拟请求一下`API`
![20200527165235130](C:%5CUsers%5Cjanki%5CDesktop%5C20200527165235130.png)
**结果失败了！**
这就是一个坑的地方，上图中的红框圈出来的地方是一个换行，而参与计算签名的JSON字符串**没有换行**，所以签名就不正确，解决方式很简单，去掉最后一个换行就可以了
![20200527165450721](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2020/image/20200527165450721.png)
查看 一下手机，需要打开应用后才能收到
![20200527165931738](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2020/image/20200527165931738.png)

**PS：**

`AppKey`和`App Master Secret`从控制台中获得
![20200527165555218](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2020/image/20200527165555218.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200527164754368.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2phbmtpbmdtZWFuaW5n,size_16,color_FFFFFF,t_70)