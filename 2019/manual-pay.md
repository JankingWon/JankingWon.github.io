---
title: 个人开发者实现实现充值和提现功能
urlname: manual-pay
tags:
  - Service
  - Java
  - Pay
copyright: true
category: Service
date: 2019-06-23 14:38:18
---

因为支付宝和微信API都不开放给个人，基于监听手机通知和广播的操作实在费神费力不讨好，于是就用手动确认代替自动到账

**方案**如下

<!-- more --> 

## 充值时

**用户**扫完页面显示的二维码后，点击确定

**前端**提供充值的`userId`，当然还需要`Cookie`认证，防止`cookie`不一致，还要提供充值的金额（闲钱币，换算:100闲钱币 = 1RMB），然后调用`/money`

**服务器**会查找数据库中该`id`的用户，然后发送邮件给管理员，管理员去看看有没有收到邮件描述的收账情况，如果有就同意充值，不然就拒绝。

![1561443481734](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/manual-pay/1561443481734.png)

一旦调用充值API，在充值记录的数据库里就会多一条记录，但是会设置`status`为`false`，只有管理员去邮件里点击确认充值的链接后才会变为`true`，这时候其余额也会真正增加

![1561443313644](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/manual-pay/1561443313644.png)

至于这个确认的链接

- 路径随机生成，理论上不存在盗刷的情况
- 不能点击两次的，点击了一次之后就会失效！

![1561443519635](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/manual-pay/1561443519635.png)

## 提现时

**用户**扫完页面显示的二维码后，点击确定

**前端**提供充值的`userId`，当然还需要`Cookie`认证，防止`cookie`不一致，还要提供提现的金额（闲钱币，换算:100闲钱币 = 1RMB），然后调用`/money` 

怎么跟充值一样的地址呢？因为可以通过传递的`json`对象的`money`字段的**正负判断是充值还是提现**

从下图中还可以看到创建问卷时自动扣余额的记录

![1561444200706](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/manual-pay/1561444200706.png)

**服务器**会查找数据库中该`id`的用户，首先看看提现金额够不够，然后看下有没有填微信或者支付宝账号（至少要填一个），确认可以提现之后，然后发送邮件给管理员，管理员去看看有没有收到邮件描述的收账情况，如果有就手动转账给邮件提供的支付宝或者微信账号（微信本身不支持匿名转账，但是我们做了），在管理员公平、负责的情况下，此时对方应该收到钱了，然后点击我已转账，该用户的记录里的`status`也会变为`true`，然后它的闲钱币就会扣除

![1561444076836](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/manual-pay/1561444076836.png)

## 源码

https://github.com/system-design2019/system-design/tree/master/backend