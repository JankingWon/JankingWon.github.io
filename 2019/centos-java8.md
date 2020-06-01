---
title: CentOS升级或安装安装JDK 8
urlname: centos-java8
tags:
  - Java
  - Linux
copyright: false
category: Linux
date: 2019-04-25 12:53:57
---

> CentOS安装JDK 8
>

<!-- more --> 

# CentOS安装JDK 8

## 准备工作

首先，更新包：

```
yum update
```

检查服务器上是否已安装旧版本的Java：

```
java -version
```

如果有旧版本的Java则移除：

```
yum remove java-1.6.0-openjdk
yum remove java-1.7.0-openjdk
```

## 下载安装JDK

前往[Oracle Java下载页面](https://link.jianshu.com/?t=http://www.oracle.com/technetwork/java/javase/downloads/index.html)，根据你的系统架构找到合适的版本。比如我的系统是Centos 6 x86，找到`jdk-8u102-linux-i586.rpm`，复制其下载地址，在服务器中下载：

```
wget --no-cookies --no-check-certificate --header "Cookie: gpw_e24=http%3A%2F%2Fwww.oracle.com%2F; oraclelicense=accept-securebackup-cookie" "http://download.oracle.com/otn-pub/java/jdk/8u102-b14/jdk-8u102-linux-i586.rpm"
```

在你下载的目录中执行rpm包安装命令：

```
rpm -ivh jdk-8u102-linux-i586.rpm
```

执行完成后会出现类似的结果：

```
Preparing...                ########################################### [100%]
   1:jdk1.8.0_102           ########################################### [100%]
Unpacking JAR files...
        tools.jar...
        plugin.jar...
        javaws.jar...
        deploy.jar...
        rt.jar...
        jsse.jar...
        charsets.jar...
        localedata.jar...
```

## 检查Java版本

现在，检查以下刚才安装的JDK版本：

```
java -version
```

如果正确安装，会出现以下结果：

```
# java -version
java version "1.8.0_102"
Java(TM) SE Runtime Environment (build 1.8.0_102-b14)
Java HotSpot(TM) Client VM (build 25.102-b14, mixed mode, sharing)
```

正如你所见，JDK 8已成功安装。

## 设置环境变量

我们可以用下面的命令设置环境变量：

```
export JAVA_HOME=/usr/java/jdk1.8.0_102/
export PATH=$PATH:$JAVA_HOME
```

测试一下环境变量：

```
echo $JAVA_HOME
```

应该输出的结果为：

```
/usr/java/jdk1.8.0_25/
```

然而，上述方法并不推荐，因为系统重启后所设置的环境变量将不复存在。为了使之永久性设置，需要在系统profile里新增路径。
在**/etc/profile.d/**路径下新建一个文件，名为**java.sh**：

```
vim /etc/profile.d/java.sh
```

写入以下语句：

```
#!/bin/bash
JAVA_HOME=/usr/java/jdk1.8.0_102/
PATH=$JAVA_HOME/bin:$PATH
export PATH JAVA_HOME
export CLASSPATH=.
```

保存并关闭文件，执行以下命令使之可运行：

```
chmod +x /etc/profile.d/java.sh
```

最后，执行以下命令来永久设置环境变量：

```
source /etc/profile.d/java.sh
```

大功告成！