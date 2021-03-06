---
title: 解决办法 | 用Ctrl + S 保存网页源码部署到自己的网站上无效的解决办法
urlname: html-save-error
tags:
  - Html
  - Js
  - 解决办法
copyright: true
category: Web
abbrlink: 12172
date: 2018-10-05 18:25:11
---

## 问题：

最近在网络上看到一个很棒的网页源码，想用`CTRL+S`复制到自己的服务器上，却怎么也跑不起来，反正就是各种动画无效，情况如下：

![1538735341216](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/html-save-error/1538735341216.png)

<!-- more --> 

## 原因：

用F12看了一下网页的加载情况，发现有很多src加载不出来，一看原来是需要加载后缀为`.下载`，我的天，怎么会有这种后缀！

![1538735179088](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/html-save-error/1538735179088.png)

打开我的网站面板看一下，果真是这个样子，而且还竟然真的存在`.下载`文件！

![1538736377891](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/html-save-error/1538736377891.png)

![1538735653614](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/html-save-error/1538735653614.png)

（其实图中那些图标为空白的也是js文件加了后缀）

## 解决办法：

第一步：把文件夹里`.下载`后缀去了就好了！

第二步：把里面html源码里面需要引用的js文件后面的`.下载`去了！

也就是说，需要引用的文件确实是`.下载`文件，也确实存在这个文件，但是无法识别为js脚本，**所以需要两个地方都改！**

这种网页保存可能是浏览器的问题，也可能是Chrome的问题（反正不是IE的问题，/滑稽）

效果如下：



> 需要注意一点哦，用Ctrl+S保存源码，最后得到的不只是html文件，还有一个文件夹保存着需要的资源，该文件夹与该网页名同名，出问题的就是这个文件夹里的资源。
