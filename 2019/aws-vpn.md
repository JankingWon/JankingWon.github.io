---
title: 免费申请亚马逊云(AWS)一年云主机使用+搭建自己的VPN
urlname: aws-vpn
tags:
  - VPN
  - VPS
  - AWS
copyright: true
category: Network
date: 2019-03-05 13:33:28
---

[TOC]

> AWS云主机的申请过程和搭建VPN过程

 <!-- more -->

>  准备：
>
> **一张信用卡**，银联VISA等都行
>
> 没了。

## 注册亚马逊云

[免费套餐注册链接](https://amazonaws-china.com/cn/free/)

之前撸谷歌云的时候就是失败在信用卡上，我申请的是**工行星座卡校园版**

这个卡申请之后会发两张（当时查看进度还以为是自己多提交了一次，有两张卡在进度中）

**一张国内用的银联卡，一张国际的VISA卡，不同卡但是同一额度**

银行卡很漂亮，就是容易刮花，现在已经面目全非了。

![20190201_122652](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/20190201_122652.jpg)

> [知乎这篇文章有申请信用卡的详细介绍](https://zhuanlan.zhihu.com/p/26943964)

但是收到之后让我沮丧的是**额度为0！**

我也不需要信用卡消费，可是一般绑定信用卡要**扣一美元**验证身份（以后会归还）啊

里面没有钱，那就当银行卡用吧，可以绑定支付宝选择信用卡还款，这里我试了一块钱，接着又还了9块，毕竟要**够一美元**才行

![qq_pic_merged_1553427559691](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/qq_pic_merged_1553427559691.jpg)

再去公众号【工行信用卡】查看就有额度了

因为我已经扣了一美元，所以只有这么多，可以证明不充钱的时候银行卡确实一分钱都没有！

![qq_pic_merged_1553427595838](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/qq_pic_merged_1553427595838.jpg)



然后去亚马逊绑定VISA的卡，它会有一个验证，但是失败了！

然后我的邮箱收到一个邮件，原文是英文，用谷歌翻译之后是这样

![1553427995805](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553427995805.png)

反正就是坏了，当时注册谷歌云也是这样，其实是里面**没有外汇！**

需要转入美元到账户里面去，理论上可以通过官方的信用卡网上银行操作，但是我没有去营业厅开通电子银行，只是在APP上**自助开通，这样是不行的**！要操作外汇只能去柜台办理电子银行才行，于是我就用银联的卡试试

好在亚马逊**支持银联！**

![1553426886092](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553426886092.png)

然后就扣了一美元，开始一年撸羊毛之旅！

## 选择VPS

### 速度对比

我之前装过新加坡的，也装过日本的VPS，但我现在重新装了**新加坡**的。

因为我在广州市，所以可能跟地区有一定影响，不过主要是给自己用，所以不用管什么全国平均访问，只要自己访问的快才是硬道理！

**日本**

![1553426521828](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553426521828.png)

**新加坡**

![1553426488760](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553426488760.png)



最近突然发现，其实可以选择**中国香港**，不过默认禁用了，得点击**我的账户**手动启用！

![1557808515719](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1557808515719.png)

### 开始

选择主机的区域是在账号界面的右上角

![1553428485410](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553428485410.png)

选择启动实例

![1553428581278](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553428581278.png)

选择系统映像，这里我选择的是`Ubuntu18.04`

> 之前我选的是最上面的额Amazon Linux2，因为linux也不是非常懂，很多地方不像ubuntu也不像centos，资料还很少，最后还是重装了

![1553428735389](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553428735389.png)

选择免费套餐的配置

其实这里就可以点击**审核与启动**了，后面都是没有可选的空间，毕竟是免费套餐

![1553429133323](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553429133323.png)

最后确认配置，修改一下**安全组**

![1553429326243](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553429326243.png)

为了方便，都设置打开，因为可以通过系统内部的端口开关来控制，外部就不要干预了

![1553429265086](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553429265086.png)

这里需要密钥对才能启动，新建一个密钥对，然后下载保存好

![1553429393837](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553429393837.png)

等待启动

![1553429435145](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553429435145.png)

### 分配弹性IP

在启动之前，建议先分配弹性IP

因为如果没有弹性IP，VPS的IP可能会发生变化，以后连接需要就经常改配置很麻烦，如果有弹性IP就相当于永久的一个IP地址

![1553429484257](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553429484257.png)

很快分配好之后，选择IP地址，右键选择关联地址

![1553429582375](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553429582375.png)

选择之前的实例和IP，应该都只有一个选项

![1553429639162](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553429639162.png)

> 注意，单独的弹性IP是不在免费套餐内的，所以需要及时绑定一个实例，如果只是分配了弹性IP，很久没有绑定实例，很可能要付费！

### 连接服务器

因为强制要用密钥连接服务器，而且我的电脑是`windows`，不好用命令行

官方推荐使用`putty`，但是我觉得UI太丑了，推荐一款超棒的`SSH`软件`terminus`

<https://www.termius.com/>

安装之后先要添加之前下载的密钥

可以使用`NotePad++`打开下载的`pem`文件

![1553430343414](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553430343414.png)

复制之后，打开`Terminus`，添加`Keychain`

![1553430392345](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553430392345.png)

![1553430443751](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553430443751.png)

添加私钥

写一个`label`标识这个密钥，然后把复制的私钥粘贴在`Private Key`里

![1553430485181](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553430485181.png)

接着回到`Terminus`主页

点击`New Host`

![1553430074978](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553430074978.png)

写好`Label`，服务器`IP`地址，并使用`SSH`，在下面的`key`选项中找到之前添加的`Key`

![1553430592381](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553430592381.png)

![1553430677132](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553430677132.png)

![1553430663101](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553430663101.png)

点击`SAVE`，就添加好了这个`VPS`

接着回到主机列表，**双击**刚刚添加的主机，会弹出一个登陆的窗口

**默认用户名是`ubuntu`，没有密码**

![1553430901972](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553430901972.png)

这是官方的连接说明

![1553430888996](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553430888996.png)

弹出是否信任，点击`YES`

![1553430956127](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553430956127.png)

然后就进入了服务器的命令行！成功！

![1553431013456](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553431013456.png)

## 安装代理服务

用过`ss`和`openvpn`，还是推荐`ss`吧，比较流行，客户端也适配的挺好！

这里以`ubuntu`为例，其他的`centos`等网上的解决方案也很多

### 安装SHADOWSOCKS

> 默认没有密码，所以`sudo`也不需要设置密码
>
> terminus的粘贴快捷键是 Ctrl+Shift+V

```bash
$sudo apt-get update 

$sudo apt-get install python-pip 

$sudo pip install git+https://github.com/shadowsocks/shadowsocks.git@master
```

第二条命令会弹出确认提示，输入`Y`

![1553431143002](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553431143002.png)

### 配置

```bash
$ sudo vim /etc/shadowsocks.json
```

复制下列的配置文件

```json
{
	"server":"0.0.0.0",
	"local_address":"127.0.0.1",
	"local_port":1080,
	"port_password":{
	"9000":"password0",
	"9001":"password1",
	"9002":"password2",
	"9003":"password3",
	"9004":"password4"
	},
	"timeout":300,
	"method":"aes-256-cfb",
	"fast_open": false
}
```

> 一个端口和密码对应一个用户
>
> vim粘贴命令： `Shift insert` 
>
> vim插入模式：`i`
>
> vim命令模式：`Esc`

输入 

```
:
wq!
```

开放端口

```bash
sudo ufw allow 9000
sudo ufw allow 9001
...
```

开启服务

```bash
sudo ssserver -c /etc/shadowsocks.json -d start
```

确认启动

```bash
ubuntu@ip-172-26-13-226:~$ /bin/ps axu | grep ssserver | grep -v grep
root     19393  0.0  3.1  55352 15540 ?        Ss   05:49   0:00 /usr/bin/python /usr/local/bin/ssserver -c /etc/shadowsocks.json -d start
```

关闭服务

```bash
ssserver -c /etc/shadowsocks.json -d start
```

查看防火墙状态

```bash
ubuntu@ip-172-31-44-56:~$ sudo ufw status
Status: active

To                         Action      From
--                         ------      ----
20,21,22,80,888,8888/tcp   ALLOW       Anywhere                  
39000:40000/tcp            ALLOW       Anywhere                  
443/tcp                    ALLOW       Anywhere                  
9000                       ALLOW       Anywhere                  
9001                       ALLOW       Anywhere                  
9002                       ALLOW       Anywhere                  
9000/tcp                   ALLOW       Anywhere                  
9000/udp                   ALLOW       Anywhere                  
9002/tcp                   ALLOW       Anywhere                  
9002/udp                   ALLOW       Anywhere                  
9001/tcp                   ALLOW       Anywhere                  
9001/udp                   ALLOW       Anywhere                  
20,21,22,80,888,8888/tcp (v6) ALLOW       Anywhere (v6)             
39000:40000/tcp (v6)       ALLOW       Anywhere (v6)             
443/tcp (v6)               ALLOW       Anywhere (v6)             
9000 (v6)                  ALLOW       Anywhere (v6)             
9001 (v6)                  ALLOW       Anywhere (v6)             
9002 (v6)                  ALLOW       Anywhere (v6)             
9000/tcp (v6)              ALLOW       Anywhere (v6)             
9000/udp (v6)              ALLOW       Anywhere (v6)             
9002/tcp (v6)              ALLOW       Anywhere (v6)             
9002/udp (v6)              ALLOW       Anywhere (v6)             
9001/tcp (v6)              ALLOW       Anywhere (v6)             
9001/udp (v6)              ALLOW       Anywhere (v6) 
```

> 其实IPV6没有用

### 连接VPN

`shadowsocks`官方网站（但是没有梯子似乎进不去）

<https://shadowsocks.org/en/download/clients.html>

`shadowsocks`官方`Github`

<https://github.com/shadowsocks>

选择平台，如`windows`版本的链接

<https://github.com/shadowsocks/shadowsocks-windows/releases/download/4.1.5/Shadowsocks-4.1.5.zip>

### 配置

安装完成之后，启动，任务栏会有一个小飞机的图标

- 先点击启动系统代理

  ![1553431874077](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553431874077.png)

- 右键->服务器->编辑服务器

![1553431973878](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553431973878.png)

- 填写下列四项，其实跟刚刚的配置文件一样的

  ![1553432056286](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553432056286.png)

- 然后就可以右键->服务器->[选择自己的服务器]

  ![1553432125369](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553432125369.png)

- 右键->系统代理模式->PAC模式

  > 通俗解释：PAC就是有的网站使用代理，有的网站直接连接
  >
  > 全局模式就是全部使用代理，即使上baidu.com也是通过非大陆的服务器中转，效率比较低，还浪费流量

  ![1553432154835](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553432154835.png)

- 但是呢，直接这样子应该是没用，因为本地PAC列表没设置

  - 右键->PAC->从GFWList更新本地PAC

  - 右键->PAC->使用本地PAC

  最好点击一下保护本地PAC

  ![1553432409394](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553432409394.png)

成功！

![1553432505203](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553432505203.png)

## 安装加速器Google TCP BBR

加速效果不错的插件

```bash
$sudo wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh
$sudo chmod +x bbr.sh
$sudo ./bbr.sh
```

安装完成后，脚本会提示需要重启 VPS，输入任意键后重启。

```bash
ubuntu@ip-172-26-13-226:~$ lsmod | grep bbr
tcp_bbr                20480  1
```

说明成功！

## 还有优惠！

为什么我之前说新加坡和日本的主机我都用过呢，因为除了这个`AWS`，还有一个**免费一个月**`VPS`的试用机会

是`AWS`的子品牌`lightsail`

<https://amazonaws-china.com/cn/lightsail/>

差不多的配置之后是这个样子

可以在这个服务器体验一下不同地区的`VPS`，对比一下

![1553432571976](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553432571976.png)

## 其他

### 安装宝塔面板

最好还是有个可视化界面操作方便一点嘛

安装命令

```bash
$wget -O install.sh http://download.bt.cn/install/install-ubuntu_6.0.sh && sudo bash install.sh
```

[官方教程](https://www.bt.cn/bbs/thread-19376-1-1.html)

安装完成后记得命令行中给的**面板地址，用户名，密码**

![1553433127826](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553433127826.png)

如果猴急关了

1. 查看面板入口：

   ```
   /etc/init.d/bt default
   ```

   

2. 关闭安全入口：

   ```
   rm -f /www/server/panel/data/admin_path.pl
   ```



建议登录之后在设置中更改安全登录入口、用户名和密码

![1553433495284](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aws-vpn/1553433495284.png)

## 参考资料

[亚马逊AWS EC2搭建shadowsocks服务](https://my.oschina.net/u/3163032/blog/1863988)

