---
2020-06-02 14:34
---

# 可能是目前最好的博客策略（长期、稳定、多平台发布）

## 概述

**利用GitHub仓库长期存放博客及图片，其他平台一键复制即可进行发布！**

## 使用的工具或资源

* [Typora](https://typora.io/)
* [PicGo](https://molunerfinn.com/PicGo/) 
* [GitHub](https://github.com/)
* [jsDelivr](https://www.jsdelivr.com/)

### 设置GitHub

在GitHub中去申请一个token，只要授予`repo`权限就好了

![](https://cdn.jsdelivr.net/gh/jankingwon/jankingwon.github.io@latest/2020/image/F6BBC329D7F14BD8B3EB154CB3D3CC7B.png)

复制保存（因为token只能复制一次，后续就看不到了）

并且单独开一个仓库用来存储**博客和图片**

![](https://cdn.jsdelivr.net/gh/jankingwon/jankingwon.github.io@latest/2020/image/20200602142613.png)

> 参考策略：
> 本人直接使用的是GitHubPage的仓库（用户名.github.io）
> 用年份作为一级目录，直接存储博客md文件，
> image作为二级目录，存储每年写的博客的图片资源
> 避免一个文件夹文件太多，同时后期也方便对每年统一回顾


### 设置PicGo

在picGo中设置GitHub图床，按照提示输入需要的信息

![](https://cdn.jsdelivr.net/gh/jankingwon/jankingwon.github.io@latest/2020/image/17BDE104FC21469D86E6A5650DFFEF62.png)

这里的自定义域名可以使用

```
https://raw.githubusercontent.com/[github用户名]/[仓库名]/master
```
不过由于 ~~众所周知~~ 未知 的原因，GitHub速度不稳定，虽然峰值其实也挺快的，只是偶尔就连不上
所以也可以使用`jsDelivr`的cdn地址（**没错，jsDelivr都不需要注册，直接使用url拼接就能实现CDN！**）
```
https://cdn.jsdelivr.net/gh/[github用户名]/[仓库名]@latest
```
`jsDelivr`是一个「快速」、「可靠」、「自动化」的用于开源数据的**免费**CDN

每个月存储了` 1920TiB` 的数据！（2020.06.02），使用人数相当多，稳定性很好，而且数据源在自己的GitHub上，是在担心数据丢失，那就加一个

> “如果本博客图片链接失效，请将[jsDelivr]链接更换为[GitHub链接]”

就可以了！

### 设置Typora

「文件」-> 「偏好设置」 -> 「图像」

「插入图片时...」的动作选择 「上传图片」，并输入picGo在本机上的位置

![](https://cdn.jsdelivr.net/gh/jankingwon/jankingwon.github.io@latest/2020/image/E3CB1E39173048D4BBA8C4C99392ED50.png)

## 写！

那么应该怎么写呢？

要记住，之前在GitHub新开的那个仓库就是用来**长期稳定**存储博客及图片的，所以需要先把那个仓库克隆下来，然后在本地博客仓库文件夹内使用Typora编辑器进行编写，因为Typora设置好了图片上传的功能，所以在本地写的时候直接粘贴图片就好啦，它会自动帮你上传。

> 可能有的同学会觉得写完还要push，还要复制很麻烦，其实把文章写好了博客仓库不用急着push，因为它就是一个备份，必要的时候在你换了设备的时候push一下就好了，否则一直留在本地都可以。
>
> 当然强迫症还是喜欢全都放在云端比较安心 hh

写好之后，一键复制到**CSDN**、**简书**、**掘金**、**segmentfault**以及自己的**个人博客网站**如**WordPress**、**Typecho**就好啦！

也就是说，GitHub上的仓库只是用来存储，但是不是用来展示的，当然也可以展示，就是比较丑，或者也可以搞个jekyll主题，反正办法很多。
主要用来展示博客的地方应该是第三方博客平台（流量多）或者自己的博客小站（可自定义、有归属感）


## 总结

**优点**
* **长期稳定**
  * 图片文章都在GitHub上，您跟我说GitHub不稳定？~~不说网络~~
* **图片速度快**
  * 全球好多好多人都在用的 jsDelivr **免费 ** **CDN**，快到飞起
* **多平台发布**
  * 因为现在简书、CSDN发布博客中的外链图片都会转存到它们自己网站上，而且有防盗链机制.....如果发布到一个新平台需要手动转存图片，很麻烦
* **个人博客网站随意开关，再也不用担心续不续费的问题了**
  
    * 因为个人博客网站就是显示的一个载体，数据库里没啥重要数据。考虑到大多数个人博客网站存活时间都不长，用了此种策略的话即使个人博客反复开开关关也不会有啥影响，写个脚本一键发布就好了，最重要的是图片不会丢 
* **写作体验好**
    * 这里不撕逼哪个编辑器是最好用的md编辑器，起码Typora所见即所得的方式比CSDN、Typecho等在线编辑器不知道要好多少！
    

**缺点**
* **修改的时候多一步**
    * 因为修改的时候需要修改一下仓库里的内容
* **跨设备编辑的话麻烦一点**
    * 因为如果编辑的话是需要修改博客仓库的，所以需要先pull下来，然后修改commit，然后push（这里一定需要push了，因为跨设备了），然后一键复制到已经发布的平台。