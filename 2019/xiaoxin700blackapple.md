---
title: 超详细！联想小新700黑苹果双系统完整教程（1）
urlname: xiaoxin700blackapple
tags:
  - 电脑技术
  - 黑苹果
  - 小新700
  - EFI分区
copyright: true
category: 电脑技术
abbrlink: 16366
date: 2018-12-21 16:17:18
---

**本教程图片特别多！即使您不是小新700的用户，也可以切身感受到装黑苹果的过程！**

## 概述

效果

![IMG_0189](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple2/IMG_0189.PNG)

基本完美，很流畅

<!-- more --> 

- WIFI无法使用，需要更换无线网卡或者用外置USB网卡（问题不大，以太网用的很棒！）

- 独立显卡无法使用（问题不大，不是用来处理图形集显完全带的动）

我的电脑配置

> 电脑型号  联想 80SH
> 操作系统  Microsoft Windows 10 企业版 (64位)
> CPU  (英特尔)Intel(R) Core(TM) i7-6700HQ CPU @ 2.60GHz(2592 MHz)
> 主板  联想 Lenovo xiaoxin 700-15ISK
> 内存  32.00 GB (   2133 MHz)
> 硬盘  
>
> - 130 GB ( SAMSUNG MZVLW128HEGR-00000)
>
> - 260 GB(Samsung SSD 840 PRO Series)
>
> 显卡  
>
> - NVIDIA GeForce GTX 950M
> - Intel(R) HD Graphics 530
>
> 显示器  京东方 BOE HF 32位真彩色 59Hz
> 声卡  Realtek High Definition Audio
> 网卡  Intel(R) Dual Band Wireless-AC 3165


## 准备文件

- **MacOS 10.13.6镜像文件**

  (这是目前安装`Xcode`的最低系统要求哦)

  > 链接：https://pan.baidu.com/s/1SdgOuX2yk8UVkbZ5VtheZA 
  > 提取码：26mh 

- **小新700EFI**

  > 链接：https://pan.baidu.com/s/1JZbHv_tcFnITmKaYGlWcDQ 
  > 提取码：mmp3 

- **TransMac安装程序**

  > 链接：https://pan.baidu.com/s/1AOFus6IesnvD5z4Vyx66Wg 
  > 提取码：h5d9 

## 写入系统镜像

右键`TransMac`，以**管理员**身份运行

![1545380960838](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/1545380960838.png)

找到要刻录的U盘，格式化

![A](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/A.png)

![B](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/B.png)

写上格式化后的磁盘名称

![C](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/C.png)

等待格式化

![D](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/D.png)

写入系统文件

![E](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/E.png)

![F](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/F.png)

选择下载的Mac系统镜像文件

![G](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/G.png)

等待进度条![H](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/H.png)

最后成功

![i](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/i.png)



## 警告

写入镜像之后会时不时蹦出这个

一定要点**取消！**

![1545383409511](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/1545383409511.png)

然后显示这个，不要担心，只是`Windows`**暂时不识别它**，不是U盘损坏了！

![1545383429274](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/1545383429274.png)

## 修改EFI分区

现在来检查一下能不能在`我的电脑`里识别U盘的EFI分区

![1545381566045](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/1545381566045.png)

> PS.这个跟Windows的系统启动盘不一样，写入系统镜像后，U盘被分成了三个区，通过磁盘管理可以看到
>
> ![1545381630698](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/1545381630698.png)
>
> - EFI是用来启动安装程序的
> - 主分区MacOS的系统文件
> - 还有一些未分配的空间，理论上可以把它右键分配成为独立于MacOS安装程序之外的空间，也就是U盘技能用来装黑苹果，还能有一个相对独立的地方装其他文件（说不定还能引导一个Windows的系统……哈哈哈）



- 如果不能显示的话需要进行下面的**显示EFI分区操作**

- 如果能显示但是如下图无法访问的话进行**修改EFI分区操作**

![1545381856711](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/1545381856711.png)

![1545381865240](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/1545381865240.png)

### 显示EFI分区操作

搜索栏中输入`cmd`，右键以管理员身份运行（右键菜单不好截图）

![1545382035179](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/1545382035179.png)

首先输入`diskpart`，打开磁盘分区工具

![1545382434728](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/1545382434728.png)

再顺序输入以下命令

```
list disk
select disk 1
list partition
select partition 1
set id="EBD0A0A2-B9E5-4433-87C0-68B6B72699C7"
assign letter x
```

![未命名图片](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/未命名图片.png)

> **注意：**
>
> 不一定是`select disk 1`，要通过显示的磁盘列表找到自己的U盘或者想要写入镜像的盘，输入它的编号
>
> 也不一定是`select partition 1`，要通过显示的分区列表找到里面的**系统**类型的分区（大概200M），输入它的编号
>
> 也不一定要分配X分区，只要是没有使用的盘符都可以

### 修改EFI分区操作

现在的话不出意外应该是能显示EFI盘符了，**但是打不开**！（上面有图有描述）

办法有很多，介绍一种简单可用的吧

[下载Explorer++](https://explorerplusplus.com/)

这是一个绿色软件，也就是不用安装直接点击运行的！



记得**右键->以管理员身份运行**它

发现可以点开了`EFI`盘（X盘）

里面有个文件夹`EFI`

`EFI`文件夹里有两个文件夹`BOOT`和`CLOVER`

![1545382616254](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/1545382616254.png)

删除它们，然后复制下载好的小新`700EFI`里面的文件到进去（就是复制到X盘里面的`EFI`文件夹里面去）

![1545382777253](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/1545382777253.png)

## 安装选项

**关机状态下**（一定要关机状态）

戳一戳小新700左边的复位孔

![](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/IMG_0130-1545414078897.JPG)

会出来一个四个选项的菜单

选择**Boot Menu**

![IMG_0131](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/IMG_0131.JPG)

然后**选择自己的安装U盘**，我这里是`USB HDD`

![IMG_0132](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/IMG_0132.JPG)

左右移动到**X标志**（mac OSX）的图标上

也就是文字为`Boot macOS install from macOS High Sierra`

**按下空格键**

![IMG_0133](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/IMG_0133-1545381693831.JPG)

默认勾选了三个选项，不用管它，移动到

`Boot macOS with selected options`

**按下回车键**

![IMG_0134](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/IMG_0134-1545381622376.JPG)

然后会出现一堆代码

等待……

![IMG_0135](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/IMG_0135.JPG)

## 开始安装

**选择中文**

![IMG_0136](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/IMG_0136.JPG)

**选择磁盘工具**

![IMG_0137](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/IMG_0137.JPG)

选择要安装`macOS`的盘，**选择抹掉**

![IMG_0138](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/IMG_0138.JPG)

名称任意取

**选择`APFS`格式**（这是新版的`macOS`采用的文件系统）

![IMG_0142](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/IMG_0142.JPG)

关闭磁盘工具，回到安装界面，**选择安装`macOS`**

![IMG_0144](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/IMG_0144.JPG)

出现如下界面

![IMG_0145](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/IMG_0145.JPG)

找到**刚刚抹掉的磁盘**，**点击安装**

![IMG_0150](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/IMG_0150.JPG)

等待……

![IMG_0155](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/IMG_0155.JPG)

## 二次安装

是因为刚刚并没有安装，只是把文件复制进去了



我的情况是，上面的进度条没有执行完就重启了，跳到了`windows`界面，没关系，继续使用U盘启动到四叶草界面

选择`Boot macOS install from Mac`

> 不一定是from Mac
>
> 这里是因为我的磁盘名字叫做Mac

![IMG_0156](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/IMG_0156.JPG)

一样的选择

![IMG_0157](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/IMG_0157.JPG)

等待……

![IMG_0161](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/IMG_0161.JPG)

**然后就好啦！**

重新进入系统就开始设置了

![IMG_0163](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/IMG_0163.JPG)

![IMG_0164](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/IMG_0164.JPG)

可以看到**确实没有无线网卡**

![IMG_0165](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/IMG_0165.JPG)

可以告一段落了

![IMG_0169](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple/IMG_0169-1545411344288.JPG)

还会有点小毛病，后面再解决！



## 参考链接

[小新吃上黑苹果13.1](https://club.lenovo.com.cn/thread-4319360-1-1.html)

[修改EFI分区](https://www.cnblogs.com/xdao/p/efi_win.html)

<!-- more --> 