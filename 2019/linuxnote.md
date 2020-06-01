---
title: Linux常规操作
urlname: linuxnote
tags:
  - Linux
copyright: true
category: Linux
abbrlink: 14292
top: true
date: 2019-01-16 18:37:55
---

> 记录`Linux`下一些常用的命令和一些软件的升级卸载等操作
>
> 因为希望每篇博客都有价值，比如怎么查看当前位置的命令都要写一篇的行为是在看不下去

<!-- more --> 

## 常用命令

### 用户类

```shell
$useradd –g sales janking –G company,employees    //创建用户janking，-g：加入主要组、-G：加入次要组

$sudo passwd root //修改root密码
```

### 定位类

```shell
$cd .. //返回上一级

$cd - //返回上次路径

$pwd //查看当前路径
```

### 网络

```bash
$netstat -tunlp | grep 端口号	//查看端口占用
```

## VIM常用命令

```
dd		//删除光标所在行
```



## 升级GCC/G++

> 适用: CentosOS 6 or 7

有时候一些新的软件需要`gcc/g++ 4.9`以上版本才行，但是`Centos`目前一般默认还是`4.8`

`devtoolset4(gcc 5.2)`及以前版本已经停止支持了，所以一般是升级到`6`或者`7`

下面的方法会同时升级`gcc`和`g++`

### 升级到 6.3

```shell
$yum -y install centos-release-scl
$yum -y install devtoolset-6-gcc devtoolset-6-gcc-c++ devtoolset-6-binutils
$echo "source /opt/rh/devtoolset-6/enable" >>/etc/profile
$source /etc/profile
```

### 升级到 7.3

```shell
$yum -y install centos-release-scl
$yum -y install devtoolset-7-gcc devtoolset-7-gcc-c++ devtoolset-7-binutils
$echo "source /opt/rh/devtoolset-7/enable" >>/etc/profile
$source /etc/profile
```

<https://www.vpser.net/manage/centos-6-upgrade-gcc.html>

## 安装Node.js

最后会安装在`/usr/npm/node-v10.15.3/`

其他版本下载地址<http://nodejs.cn/download/>

> 安装node需要先把gcc升级到4.8以上

```shell
$cd /usr
$mkdir npm
$wget https://npm.taobao.org/mirrors/node/v10.15.3/node-v10.15.3.tar.gz
$tar zxvf node-v10.15.3.tar.gz 
$cd node-v10.15.3/
$./config
$make && make install //这个要好久
```

<https://www.npmjs.com/package/xinui>

## 升级或安装JAVA

现在`Oracle`要注册了，以前可以直接复制链接下载的

这是`java8`的下载[地址](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)，`Centos`选择`rpm`格式

用`wget`(带`header`参数)或者直接把文件下载下来，再用宝塔面板上传到一个目录，比如`usr/java`

```shell
$rpm -ivh jre-8u211-linux-x64.rpm 
$java -version //其实此时就可以用java了
```

添加环境变量

```shell
$ vim /etc/profile.d/java.sh //新建配置
```

写入

```
VA_HOME=/usr/java/jre1.8.0_211-amd64/
PATH=$JAVA_HOME/bin:$PATH
export PATH JAVA_HOME
export CLASSPATH=.
```

保存

```shell
$source /etc/profile.d/java.sh
```

<https://www.jianshu.com/p/848b06dd19aa>

## 宝塔面板

[文档](http://docs.bt.cn)

### 安装

[安装教程](<https://www.bt.cn/bbs/thread-19376-1-1.html>)

**Linux面板6.9.4安装命令**：使用SSH 连接工具（查看使用方法），挂载磁盘后（查看），根据系统执行框内命令开始安装（大约2分钟完成面板安装）
`Centos`安装命令：

```shell
$yum install -y wget && wget -O install.sh http://download.bt.cn/install/install_6.0.sh && sh install.sh
```

`Ubuntu/Deepin`安装命令：

```shell
$wget -O install.sh http://download.bt.cn/install/install-ubuntu_6.0.sh && sudo bash install.sh
```

`Debian`安装命令：

```shell
$wget -O install.sh http://download.bt.cn/install/install-ubuntu_6.0.sh && bash install.sh
```

`Fedora`安装命令:

```shell
$wget -O install.sh http://download.bt.cn/install/install_6.0.sh && bash install.sh
```

### 登录

```shell
$/etc/init.d/bt default   //查看面板入口
$rm -f /www/server/panel/data/admin_path.pl  //关闭入口验证
$rm -f /www/server/panel/data/domain.conf  //关闭域名登录
```


