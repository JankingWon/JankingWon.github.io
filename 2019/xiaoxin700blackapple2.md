---


title: 超详细！联想小新700黑苹果双系统完整教程（2）
urlname: xiaoxin700blackapple2
tags:
  - 电脑技术
  - 黑苹果
  - 小新700
copyright: true
category: 电脑技术
abbrlink: 31726
date: 2018-12-22 00:57:01
---

**这次来讨论黑苹果简单的优化问题！**

## 准备文件

- **Disk Genius破解版**

  > 链接：https://pan.baidu.com/s/1k-A9MfZ7Gghrb54YT8b7UA 
  > 提取码：3ais 

- **带显卡驱动EFI**

  > 链接：https://pan.baidu.com/s/1yQlxXKnlxasfg0Hzup3ZPQ 
  > 提取码：cuiw 

- **EasyUEFI**

  > 链接：https://pan.baidu.com/s/1ExKErTnfRZBEJFf2iclAJg 
  > 提取码：24jj 

- **Clover Configurator5.3**

  > 链接：https://pan.baidu.com/s/1nlBfiZG_IFooG8DuM-jX0w 
  > 提取码：5z9h 

<!-- more --> 

## 注入显卡驱动

上篇教程安装之后的黑苹果勉强能使用，但是总是会出现卡顿的情况，而且有时候显示也不流畅，因为上次的EFI文件还没有显卡驱动



首先要选中你打算安装黑苹果的**硬盘**

注意**是硬盘**不是分区

然后右键选择

**转换分区表类型为`guid`格式**

- 如果是灰色那么就是正确的

- 如果不是就要点转换

> 注意：转换`guid`的话所在的硬盘的当前任何系统都会损坏，必须要重新安装才可以，这一点要注意了。

![1545412218710](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple2/1545412218710.png)

> 理论上上面这一步是不需要做的，如果不是GUID格式的，那么后面还有更多事
>
> 参考这个教程
>
> > 链接：https://pan.baidu.com/s/1MzbgtChKdxeo32NNaVgGdw 
> > 提取码：89h0 
>
> 然后根据之前的博客重装一遍macOS

先把ESP分配个磁盘号

![1545413355927](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple2/1545413355927.png)

> 老实说这一步我也不怎么记得了
>
> 反正后面应该跟我接下来的界面一样就好了

然后把下载下来并解压的**BOOT和CLOVER**文件夹

放进ESP分区中的EFI文件夹中

**替代**原来EFI里面的BOOT和COVER文件夹

![1545413086186](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple2/1545413086186.png)

## 设置开机选择菜单

按照下列路径打开`EasyUEFI`

![1545413406560](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple2/1545413406560.png)

点击添加

![1545413446006](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple2/1545413446006.png)

选择**Linux或其他操作系统**

再选择`mac`磁盘的那个`EFI`分区（上一步把文件弄进去的那个分区）

![1545413535585](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple2/1545413535585.png)

点击**浏览**

选择该分区下的`CLOVERX64.efi`文件

点击确定

![1545413622529](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple2/1545413622529.png)

通过移动顺序，把刚刚设置的启动项移到第一个，**完成**！

以后开机就有像安装时候一样的四叶草选择系统菜单了！

![1545413695386](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple2/1545413695386.png)

## 解决外接显示器问题

到这的话，注入了新的显卡驱动后

外接显示器会有问题，我的情况是显示器只显示背景，笔记本屏幕黑屏！

不着急……

![IMG_0171](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple2/IMG_0171.JPG)

下载文件**Clover Configurator5.3**

打开之后，选择安装MAC的那个磁盘分区，点击**挂载（Mount Partition）**

![IMG_0196](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple2/IMG_0196.PNG)

然后**点击Open**

![IMG_0190](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple2/IMG_0190.PNG)

找啊找，找到文件`config.plist`

> 这些文件好像是不自动对齐的，所以如果窗口过小可能看不到……

![IMG_0194](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple2/IMG_0194.PNG)

右键，**用Clover Configurator打开**
![IMG_0191](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple2/IMG_0191.PNG)

又跳出来一个`Clover Configurator`，点击左边的`SMBIOS`

然后点击右边显示屏下面的选择按钮

![IMG_0192](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple2/IMG_0192.PNG)

**选一个`IMAC`型号的显示器**

注意：最好选用同时代的`CPU`，比如我的电脑是`i7-6700HQ`，那我就选择`i7-8700K`

![IMG_0193](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple2/IMG_0193.PNG)

直接点击关闭就可以了！

> 此操作有风险，所以最好备份好之前的配置，或者直接记录好之前选择的型号

## 最后欣赏一下

![IMG_0189](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple2/IMG_0189.PNG)

![IMG_0187](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple2/IMG_0187.PNG)

![IMG_0188](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/xiaoxin700blackapple2/IMG_0188.PNG)

<!-- more --> 