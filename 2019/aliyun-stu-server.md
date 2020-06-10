---
title: 从申请阿里云云翼计划学生服务器到部署SpringBoot项目
urlname: aliyun-stu-server
tags:
  - AliYun
  - Server
copyright: true
category: Service
date: 2019-05-18 15:44:46
---

## 购买

> 注意，前提要进行实名认证和学生认证

[云翼计划链接](<https://promotion.aliyun.com/ntms/act/campus2018.html>)

### 产品选择

进去之后有两款产品选择

![1558165729691](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-stu-server/1558165729691.png)

如何选择它们呢？

- 轻量用户服务器
  - 流量有限，但是带宽大
  - 适用于小型网站，能显著提高访问体验
- 云服务器ECS
  - 不限制流量，带宽小
  - 适用于略大型网站或者流量大的网站

我这里选择的是轻量应用服务器，我感觉流量应该也用不完，主要是这个带宽太良心了，某讯云只有`1M`可以选......

### 预装镜像

一种是**应用镜像**

- `WordPress 4.8.1`

  - 一个强大的`php`博客系统，有丰富的后台管理和主题设置

- `LAMP 6.1.0`

  - 预装了一组自由软件`Linux`，`Apach`，`Mysql`，`PHP`，用于建`PHP`网站可以选择

- 宝塔`Linux`面板 `5.2.0`

  - **强烈建议**，一个可视化的网站管理面板，什么`WordPress`，`LAMP`啥的也都可以用它来装
- 但是版本不是最新，后续可以在线升级
  

![1558242151499](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-stu-server/1558242151499.png)

- `ASP.NET`

  - 不是很了解，应该是`.NET`网站的运行环境

![1558166256044](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-stu-server/1558166256044.png)

还有一种是**系统镜像**

既然都买了服务器，这三个系统基本都很清楚了

但是还是推荐`CentOS`，基本就是为服务器而生的，也是鸟哥(`Linux`私房菜作者)推荐的系统

![1558166650574](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-stu-server/1558166650574.png)

我这里选的是`Centos`镜像，因为方便自己来管理

### 时间选择

不用着急买很久的，先买一个月，阿里云的政策还是比较好的

只要学生身份没到期，服务器没到期，都是可以进行续费的，不像某讯云，只能续费两次，然后服务器没了就是真的没了

![1558166885670](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-stu-server/1558166885670.png)

## 第一件事

付完款之后，服务器准备一段时间，然后点击控制台就可以进入命令行了

![1558165544216](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-stu-server/1558165544216.png)

那么有了一个服务器第一件事是干什么？

升级内核？装环境？装面板？

都不是！应该先输入一个命令

```shell
$rm -rf /*
```

为了提高成功率，先切换到`root`账号

```shell
$sudo su root
```

这里选择的是阿里云自带的网页版的命令行（因为毕竟待会就要重装了。。。。）

下图超长时间，而且还没录完....

![GIF](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-stu-server/GIF.gif)

最后看到，除了`ls`，`pwd`，`cd`等命令，别的命令基本失效了

其实执行过程中很多都提示权限不够无法删除，说明还是`Centos`有一些机制保护的

![1558168207583](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-stu-server/1558168207583.png)

好了，然后重装系统吧

![1558168444459](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-stu-server/1558168444459.png)

## 放行端口

看需要放行吧，起码`22`用来`ssh`得要开放

![1558172104445](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-stu-server/1558172104445.png)

## 连接服务器

不对啊，刚刚没让我输入`root`密码啊，那`root`密码是多少？

目测应该是没有密码

可以用命令行重置密码，也可以点击服务器管理的重置密码

![1558168925570](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-stu-server/1558168925570.png)

### 创建密钥

这是一个比较安全的方式登录服务器，即使别人获取了密码也登录不了

![1558169623510](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-stu-server/1558169623510.png)

然后输入密钥名字，会自动生成并且自动下载，此时要把这个下载文件保存好，因为以后再也找不到这个密钥了

![1558169682577](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-stu-server/1558169682577.png)

这里我是用`Termius`[连接](https://termius.com/)

然后把下载的目录里面的内容（也就是私钥）复制到`Termius`的`KeyChain`里

![1558170079812](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-stu-server/1558170079812.png)

然后点击`Hosts`，新建一个阿里云服务器的连接，其`IP`地址在服务器管理哪里看得到，填公网`IP`！

下面选SSH，用户名为`root`或`admin`，然后选`keys`，找到之前添加的`key`

> admin是阿里云自己新建了这么一个用户，没有密码

![1558170550037](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-stu-server/1558170550037.png)

进去之后还是先创建一个非`root`账户比较安全

```shell
$useradd janking
$passwd janking
$su janking
```

以后就可以用这个账户登陆了

顺便说一下，阿里云提供的`admin`账号没有密码，是不能通过`ssh`登录的，也要通过`passwd`修改密码

![1558171184252](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-stu-server/1558171184252.png)

## 安装面板

技术太菜，还是得要管理面板~~Carry~~辅助才行

升级内核

```shell
$su
$yum update
安装
$yum install -y wget && wget -O install.sh http://download.bt.cn/install/install_6.0.sh && sh install.sh
```

安装完成后会告诉**面板地址**和**账号密码**

![1558172586489](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-stu-server/1558172586489.png)

进入面板会提示安装套件，比如`LNMP`

![img](https://box.kancloud.cn/3020066b5492ac8de154a534c4f36878_660x430.png)

> 根据之前的经验，22端口容易被攻击，所以最好还是把ssh端口改了

进入安全，更改`ssh`端口

![1558173558885](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-stu-server/1558173558885.png)

然后要记得放行新端口以及在阿里云服务器控制台防火墙放行这个端口

之前的`ssh`会话也要更改端口重新连接

## 建立网站

点击添加**站点**，填写**域名**

> 域名请查看这篇文章[阿里云邮件服务绑定域名](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-email.html)
>
> 前提是域名要解析到服务器的这个IP地址

![1558173981943](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-stu-server/1558173981943.png)

创建完成后，默认只有一个`index`页和`404`页

![1558174128561](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-stu-server/1558174128561.png)

访问效果如图

![1558174156910](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-stu-server/1558174156910.png)

尝试修改`index.html`文件也可以实时看到不同的网页

所以，后来只要在这个文件夹里放上`html`文件和一些静态资源，就可以通过域名加路径访问到了！

所以这个就没什么说的，下面看下怎么把`SpringBoot`项目放上去

## 安装JAVA环境

默认是没有`java`的，需要安装`jre`或者`jdk`

这是`java8`的下载[地址](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)，`Centos`选择`rpm`格式

> 因为`Oracle`现在要登录才能下载`java`啦，不能直接`wget url`下载，需要一些`cookie`等认证的`header`才行，比较麻烦，还不如下载手动上传

用`wget`(带`header`参数)或者直接把文件下载下来，再用宝塔面板上传到一个目录，比如`usr/java`

然后执行

```shell
$rpm -ivh jre-8u211-linux-x64.rpm 
```

## 部署SpringBoot

使用`maven`命令`package`打包生成`jar`包

> 在IDEA中直接点击就可以，当然也可以用命令行输入`mvn package`

![1558174829309](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-stu-server/1558174829309.png)

位于`/target`目录下会生成类似`swsad-0.0.1-SNAPSHOT.jar`文件名的文件

然后登录面板，把本地生成的`jar`压缩包上传至网站目录下

![1557644311214](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-stu-server/1557644311214.png)

然后输入命令

```shell
$ touch log.txt
//linux 持续运行，输出日志到log.txt，每次运行日志会清空
$ nohup java -jar swsad-0.0.1-SNAPSHOT.jar > log.txt &
$ cat log.txt //查看运行输出

要重新启动的话需要把后台运行的进程给kill掉
$ ps -def | grep jar
$ kill 进程号
```

> 注：此处可用宝塔面板的命令行入口，也可以用`Termius`使用`ssh`连接，只是要进入正确的目录
>
> 如 `cd /www/wwwroot/api.timoney.xyz`

现在就可以通过`IP`地址加端口号如<http://47.106.191.222:8080/>访问了

> 首先要放行8080端口哦

## 绑定域名

但是如何把这个`8080`端口的的服务绑定域名呢，或者说把它映射到一个域名的`80`端口去，让它通过类似`api.example.com`这样的方式访问

其实很简单，修改一下网站的配置文件就好了

点击网站的设置，添加以下配置

    location / {             
           proxy_pass http://localhost:8080/;              
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;              
           proxy_set_header X-Forwarded-Proto $scheme;              
           proxy_set_header X-Forwarded-Port $server_port;         
    }
![1558177349987](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-stu-server/1558177349987.png)

> 最好把绿色部分配置删掉，让网站指读取spring boot的内容

然后就可以用如`api.timoney.xyz`访问`api`了

![1558177514699](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-stu-server/1558177514699.png)

> `chrome`提示不安全，因为不是`https`协议，只要申请个免费的`ssl`证书就好了，不过这样也不影响