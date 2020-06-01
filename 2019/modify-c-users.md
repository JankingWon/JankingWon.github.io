---
title: win10系统修改c盘user文件夹下的计算机名称
urlname: modify-c-users
tags:
  - Windows
copyright: true
category: Windows
abbrlink: 5590
date: 2018-12-09 17:25:57
---

[原文链接](https://www.zhihu.com/question/51241293/answer/125148050)

# 不修改用户名

有些时候，`C:\Users\`路径有中文的话，很多软件搞不好因为这个问题都会莫名其妙的报错，尤其是那些以python和R语言为开发语言的软件(`Anaconda`, `ArcGIS`, `QGIS`, `RStudio`,`CCS`等)，令人抓狂毫无脾气。对于不想折腾的人来说，最简单的方法是直接修改临时文件夹位置，参考[此链接](https://www.zhihu.com/question/51241293/answer/134845206)即可解决大部分的应用报错
https://www.zhihu.com/question/51241293/answer/134845206

<!-- more --> 

# 修改用户名

**方法一： 新建新账户，并做账户数据同步与转移 【会损失一部分数据，但是操作简单】**

**方法二： 重命名帐户名，修改注册表等相关信息 【可以做到不损失数据，但是操作有难度和风险】**

## 方法一： 新建新账户，并做账户数据同步与转移

- 如果还是想修改`User`下的用户名，在`Win10`下，最优雅正确的做法是这样滴（原谅我原装英文版的操作系统，看位置对应点吧）：打开设置（或控制面板）在帐户（也叫做用户帐户）中新建一个帐户，这个帐户必须是是本地帐户。新建时填写的用户名便是 `User` 目录下的显示名称（浏览器中点击图片看高清大图）。

![](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/modify-c-users/1.jpg)



- 紧接着提权到管理员权限。



![](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/modify-c-users/2.jpg)

- Win10 2017年2月更新后，单纯的新建本地账户不会出现新文件夹，要先注销然后登陆那个本地账户才能出现！！！登陆之后User就有这个文件夹了，这个不要担心，先进行下一步：点最后一个菜单，将当前帐户（旧帐户）的自动同步设置开启！开启！开启（Win 8、8.1、10 都有此功能，而且默认的是开启的，**下面的图是 错！误！示！范！**）

![](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/modify-c-users/3.jpg)



- 当前帐户切换到本地。


![](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/modify-c-users/4.jpg)





- 注销，登陆刚刚创建的本地账户。在新创建的帐户中登陆微软帐号，开启同步设置，这样就好了。由于有些用户数据如桌面、我的文档，`AppData`还残留在旧的账户里，所以可以将旧账户里面的对应数据直接复制到新账户里即可简单点就是，新建本地帐户，填写想要的用户名，在新帐户中登陆微软帐号，同步设置能节省一大堆麻烦的事情。简单靠谱高效


## 方法二：重命名帐户名，修改注册表等相关信息

### 创建还原点，备份注册表

**千万不可跳过这一步**

**千万不可跳过这一步**

**千万不可跳过这一步**

只有在正确的备份了**注册表**和**还原点**之后，才可以进行之后的工作redg

`Win+R`，输入`regedit`，点击确定打开注册表编辑器

选择文件->导出生成备份文件，还原的时候导入就好了

![1544348908167]http://blog.janking.cn/post/modify-c-users/1544348908167.png)

### 新建本地管理员账户

控制面板 -> 用户账户 -> 管理其他账户 -> 在电脑设置中添加新用户

![](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/modify-c-users/5.jpg)



将其他人添加到这台电脑 -> 我没有这个人的登录信息 -> 添加一个没有`Microsoft`账户的用户 ->输入用户名（**英文哦**）和密码，然后修改该账户的类型为管理员（`Administrator`）。

然后注销要修改的账户，登录新账户。



### 改用户名

1.在新账户中，首先`Win+R`，运行对话框中键入**netplwiz** 。单击**确定。**

2.在用户名称部分中，选择要更改的名称，然后单击用户名称**属性。**

3.在属性窗口，在用户名字段中提供所需的用户名称。然后单击**应用**紧接着**确定。**



![](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/modify-c-users/6.jpg)

​	4.打开`C:\Users\`，重命名账户名（如果没有获取管理员权限，则不会出现重命名这个选项给你）

然后你重命名为`HelloWorld`时，可能会弹出文件夹被其它程序占用的错误



​	5.结束占用的进程

ctrl+alt+del 呼出任务管理器 -> 性能 -> 打开资源管理器 -> CPU -> 关联的句柄 ->搜索句柄

![](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/modify-c-users/7.jpg)

结束它，然后重新重命名，就OK了。

​	6.修改注册表里的用户名

`Win+R`，输入`regedit`，点击确定。在注册表编辑器中定位到以下路径：

`HKEY_LOCAL_MACHINE/SOFTWARE/Microsoft/Windows NT/CurrentVersion/ProfileList`

在`ProfileList`文件夹下，分别点击名字为较长的字符串的文件夹，查看窗口右侧的`ProfileImagePath`键的内容，凡是`S-1-5`开头的文件夹，全都要翻，找到所有路径为`C:/Users/中文用户名` 的键。

![](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/modify-c-users/8.jpg)

双击改成英文用户名HelloWorld。

重启电脑

### 注册表批量替换

不出意外的话，现在可以成功进入系统了，而且C:\Users\已经变成要修改的英文文件了，但是

**到这里还没有完！！！！**

**到这里还没有完！！！！** 

**到这里还没有完！！！！**

因为注册表里还依然存在大量的（1000条左右）C:\Users\中文用户名  的键值，这时候就需要把所有的注册表键值给改回来。

win10自带的注册表修改器已经满足不了批量替换的需求了，需要求助于别的工具

（此处不是广告）
Registry Workshop  [Download Free Trial Software](https://link.zhihu.com/?target=http%3A//www.torchsoft.com/en/download.html)

这个软件提供了方便的批量替换功能，官网下载软件，安装。

然后安装完的第一件事情

文件-备份到

**进行注册表的二次备份！！！**

之后就是`ctrl+F`

![](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/modify-c-users/8.jpg)

出来如下结果

![](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/modify-c-users/9.jpg)

然后`ctrl+R`

![](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/modify-c-users/10.jpg)

把含有“中文用户名”的地方，替换成对应的英文即可

**不要直接查找“中文用户名”，替换成“HelloWorld”，要注意加一下路径的前缀！**

然后再重启一下！

<!-- more --> 