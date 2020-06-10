---
title: 解决办法 | 如何优雅地在Hexo框架博客中插入图片
urlname: hexopicture
tags:
  - Hexo
  - Markdown
  - 解决办法
copyright: true
category: Hexo
abbrlink: 51951
date: 2018-10-06 21:13:59
---



## 如何用Hexo写博客

在本地`Hexo`博客的目录下（子目录也行），打开`powershell`，输入`Hexo new “helloworld”`, 就可以新建一个空标题（指的是博客文章）和名称（指的是md文件）都为`helloworld`的`md`文件了，最好不要自己手动新建一个`md`文件，最起码对图片插入不友好，后面会提到

## MarkDown编辑器推荐

`Typora` windows平台就它了，不用挑了！

![1538833684287](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/hexopicture/1538833684287.png)

## 研究插入图片的方法

### 1.直接把CSDN写的博客源码复制过来？

**行不通的！**

- CSDN设置了权限，没有登录CSDN账号的浏览器访问不了CSDN链接的图片
- 有水印有水印有水印！

### 2.找个云床？

**我不喜欢**

而且我也没试过，因为

- 得找个靠谱的云床，不然万一突然倒闭啥的，那之前写的博客不就无图无真相了？
- 而且博文就应该跟图片共生共灭，图片无法加载的博客实在不愿意看！

### 3.放在本地跟Hexo一起上传到Github？

**很棒，但是需要克服一些问题**

- **图片怎么整理？**

  这个很容易想到放在跟博客名称相同的一个同级文件夹中最稳不过了！主要是这个是`Hexo`自带的功能，打开`Hexo`的`_config.yml`，修改下面这个地方，表示创建新文章时自动创建一个同名的文件夹用来放资源，这也就是为什么推荐用命令行新建文章的原因。

  ![1538833077471](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/hexopicture/1538833077471.png)



- **图片怎么插入？**

  当然指的不是直接用手把图片塞到文件夹里，再用本地路径引用它的名字……

  这个就是`Typora`的神奇之处了，你可以用QQ截图之类的工具把截图放到剪切板里，在编辑文章的时候粘贴就好了，而且可以设置它的图片保存位置，如下图

  ![](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/hexopicture/settypora.png)

  要注意选择优先**使用相对路径**，不然就会变成

  ![](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/hexopicture/errset.png)

  正确的应该是这样

  ![1538833642227](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/hexopicture/1538833642227.png)

  不过呢，这样还是不能直接发布出去，因为我们的Hexo默认使用**绝对路径而不是相对路径**，发布出去的情况是这样的

![1538834227183](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/hexopicture/1538834227183.png)

![1538834309726](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/hexopicture/1538834309726.png)

生成的网页中的图片加载不出来，因为它指向post目录中的图片名称，但是根本没有这个图片啊！因为它是放在文件夹里面的啊！

**这里其实最开始是成功的，我也不明白为什么后来就不成功了！**

**明明都是用的相对链接嘛，之前编辑插入的时候前面已经有文件夹名了，我觉得可能是某些设置我不小心改了，也有可能是`Hexo`的`bug`**

> 下面是之前生成的链接是没有`/post`路径的情况，就是及类似于把编辑器中的图片路径直接加载了域名后面的思考……似乎现在不是了

- ~~那么可以注意到刚刚的配置文件中有选项是否使用相对路径，为什么不能选呢？**当然不能啦**（应该说我觉得这样不好），它里面的很多内部逻辑都是用绝对路径使用的，我试了一次，最后发现点不了博客的菜单，因为之前默认是/tag，/category等等这样的链接，现在用相对链接找不到某个文章目录下的这些文件，我觉得这样后期修改太麻烦了！~~

  ![1538833796822](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/hexopicture/1538833796822.png)

  > ~~真的不明白为什么在Hexo上相对链接绝对链接不能兼容使用，像linux一样啊~~

### 最后解决：

办法很笨，就是每次写完博客之后把所有的图片链接前面加上`/post`路径就能保证生成的网页中图片链接正常引用了！

![1538834897233](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/hexopicture/1538834897233.png)

<!-- more --> 