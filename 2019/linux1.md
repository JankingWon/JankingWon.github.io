---
title: linux学习笔记（一）| CentOS的安装与网络配置
urlname: linux1
tags:
  - Linux
copyright: true
category: Linux
abbrlink: 47897
reward: true
date: 2019-01-15 02:01:54
---

## 下载Linux镜像

我这里使用的是CentOS（跟鸟哥的Linux私房菜相配合）

而且是Minimal版本：http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1810.iso

也可以选择阿里巴巴的镜像：https://opsx.alibaba.com/mirror

<!-- more --> 

## 安装虚拟机

- 打开[VMWare](https://www.vmware.com/)，点击新建虚拟机

![1547490437732](http://blog.janking.cn/post/linux1/1547490437732.png)

- 选择下载的镜像位置

![1547489329469](http://blog.janking.cn/post/linux1/1547489329469.png)

- 选择虚拟机文件存放的位置

![1547490415647](http://blog.janking.cn/post/linux1/1547490415647.png)

- 一直按照默认设置点击下一步，看到安装的界面出来，选择`Install CentOS 7`

![1547489436887](http://blog.janking.cn/post/linux1/1547489436887.png)

- 然后输入Enter键

![1547490524816](http://blog.janking.cn/post/linux1/1547490524816.png)

- 出来了图形化的安装界面

- 选择语言（建议选择英语）

![1547490630576](http://blog.janking.cn/post/linux1/1547490630576.png)

- 这时候发现安装不了，因为需要把**带警告标志的设置**重新设置一遍

![1547490747592](http://blog.janking.cn/post/linux1/1547490747592.png)

- 只需要点击左上角的`Done`就好了，一般都是默认设置好

![1547490843016](http://blog.janking.cn/post/linux1/1547490843016.png)

- 可以点击`Begin Installation`开始安装了

![1547490944322](http://blog.janking.cn/post/linux1/1547490944322.png)

- 还需要设置ROOT用户的密码以及新建一个正常使用的用户

  > 其实可以不用创建新用户，设置了密码之后右边的警告就消失了，但是最好还是要一个一般的用户，不然一直使用root用户容易造成不可恢复的错误

![1547490997048](http://blog.janking.cn/post/linux1/1547490997048.png)

下面的进度条走完之后，点击`Reboot`重启虚拟机

![1547492257932](http://blog.janking.cn/post/linux1/1547492257932.png)

## 基本配置

### 登陆用户

第一次登陆会询问您登陆的用户

可以输入之前新建的用户，也可以输入`root`

![1547492492612](http://blog.janking.cn/post/linux1/1547492492612.png)

输入密码之后就进入到了主界面！

![1547492670626](http://blog.janking.cn/post/linux1/1547492670626.png)

没错，这就是`CentOS`的主界面，这并不是`Windows`启动之前的那种选项，而是已经完全启动了这个系统！

因为上一步安装之后的CentOS是没有图形界面的，也就是**只有黑乎乎的命令行！**

### 设置NAT

首先**右击**点击右下角

![1547491764006](http://blog.janking.cn/post/linux1/1547491764006.png)

设置成`NAT`模式

![1547491841571](http://blog.janking.cn/post/linux1/1547491841571.png)

然后在`Windows`主机的`Power Shell`（或者`Cmd`) 中输入

```
ipconfig
```

![1547491906629](http://blog.janking.cn/post/linux1/1547491906629.png)

![1547491967822](http://blog.janking.cn/post/linux1/1547491967822.png)

记下`VMnet8`的`IPv4`地址

> 192.168.171.1

打开编辑->虚拟化网络编辑器

![1547496193357](http://blog.janking.cn/post/linux1/1547496193357.png)

选择NAT->更改设置

![1547496254429](http://blog.janking.cn/post/linux1/1547496254429.png)

选择NAT模式->NAT设置

![1547496316450](http://blog.janking.cn/post/linux1/1547496316450.png)

记录下网关`IP`

我这里是

```
192.168.171.2
```

![1547499372712](http://blog.janking.cn/post/linux1/1547499372712.png)

### 设置IP

输入命令

```
nmtui
```

选择编辑网络

![1547491611588](http://blog.janking.cn/post/linux1/1547491611588.png)

选择唯一的一个以太网卡`ens33`

![1547491649244](http://blog.janking.cn/post/linux1/1547491649244.png)

- 设置配置为`Manual`

  - 设置`Addresses`为之前的`VMnet8`的同一网段`IP`（就是192.168.171	.多少多少）

- 设置`Gateway`为之前的`VMnet8`的`IP`（就是**192.168.171.2**）

  

  > 这里有很多误区！
  >
  > 很多教程是填Windows下查看的VMnet8的IP地址(192.168.171.1)
  >
  > 其实应该是**NAT设置里面的网关(192.168.171.2)
  >
  > 

- 设置`DNS`为`8.8.8.8`（是Google提供的免费DNS服务器的IP地址）或`114.114.114.114`

  [选择一个可用的公共DNS](https://www.zhihu.com/question/32229915)

![](http://blog.janking.cn/post/linux1/1547499079705.png)

### 激活网卡

设置之后返回选择`Activate a connection`

![1547492337404](http://blog.janking.cn/post/linux1/1547492337404.png)

按右键选择右边的`Activate`

> 我这里显示的是Deactivate，就是说已经激活了
>
> 如果显示的是Activate，就是还没有激活

![1547492369368](http://blog.janking.cn/post/linux1/1547492369368.png)

> 即使网卡已经激活了，每次设置完网卡的信息
>
> 最好也要把它先禁用再激活一次（就是连续按两次）

### 验证网络

再次回到命令行，输入

```
ip addr
```

确认`ip`已经配置成功

![1547492805138](http://blog.janking.cn/post/linux1/1547492805138.png)

输入

```
ping baidu.com
```

出现下面，说明网络连通

![1547496096481](http://blog.janking.cn/post/linux1/1547496096481.png)

如果出现这种情况

![1547492899895](http://blog.janking.cn/post/linux1/1547492899895.png)

说明需要重新配置，一般是DNS错误或者网关错误

## 思考

正确的结论是，虚拟机里面的网关地址一定要跟这个`VMWare NAT`里面的网关一致（上图）

至于刚刚记下的`Windows`里面的`VMnet8`网卡其实没有起到关键作用，它只是作为连接虚拟机的一个桥梁而已

单独写出来是为了**区分其他忽视了这个原理的教程**！

您甚至可以把它禁用掉，虚拟机**仍然能够连接外网**！只是**主机连接不上虚拟机**！

[了解更多](https://www.linuxidc.com/Linux/2017-09/147080.htm)

