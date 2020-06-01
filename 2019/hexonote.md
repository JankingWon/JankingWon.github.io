---
title: Hexo+NexT搭建博客笔记
urlname: hexonote
tags:
  - Hexo
copyright: true
category: Hexo
date: 2019-01-17 01:37:09
---

# 前言

之前的博客感觉胡乱抄别人的“优化”代码，现在已经感觉理不清里面的结构了，所以一气之下还是重新安装了

<!-- more --> 

# 安装HEXO

官网说可以这样子安装

```bash
npm install hexo-cli -g
hexo init blog
cd blog
npm install
```

# 安装主题

之前的Next仓库是 https://github.com/iissnan/hexo-theme-next

新版本的Next已经移动到这里了 https://github.com/theme-next/hexo-theme-next

旧的仓库几乎已经不更新了



只要用最简单的`git`命令就可以安装了

```git
git clone https://github.com/theme-next/hexo-theme-next themes/next
```

# 基本配置

配置的是主题下的`_config.yml`文件，就是`blog\themes\next`路径下的配置文件

## 网站图标

[网站图标生成网站](https://realfavicongenerator.net)

只要把`png`图片放进去（尺寸不能太大）

就可以生成一套图标（苹果手机桌面图标，网页图标，win磁贴图标等等）

![1547669514508](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/hexonote/1547669514508.png)

下载之后放到`blog\source\my_images`文件夹中

```yaml
favicon:
  small: /my_images/favicon-16x16.png
  medium: /my_images/favicon-32x32.png
  apple_touch_icon: /my_images/apple-touch-icon.png
  safari_pinned_tab: /my_images/safari-pinned-tab.svg
  android_manifest: /my_images/site.webmanifest
  ms_browserconfig: /my_images/browserconfig.xml
```

## 网站脚注

```yml
footer:
#建站时间
  since: 2018
#作者头像并且是动画效果
  icon:
    name: user
    animated: true
    color: "##66CDAA"
#显示版权作者
  copyright: Janking
#不显示Hexo
  powered:
    enable: false
    version: false
#不显示主题和版本
  theme:
    enable: false
    version: false
#显示备案号
  beian:
    enable: true
    icp: 粤ICP备18059833号-3
```

## 版权声明

[具体详情](https://creativecommons.org/share-your-work/licensing-types-examples)

```yml
creative_commons:
  license: by-nc-sa
  sidebar: true
  post: true
```

## 代码块

```yml
codeblock:
  # 自定义边框半径，默认是1
  # 值越大弧度越大
  border_radius: 6
  # 右上角显示复制按钮
  copy_button:
    enable: true
    # 显示复制结果
    show_result: true
```

## 分享

```yml
needmoreshare2:
  enable: true
  postbottom:
  #文章底部
    enable: false
    options:
      iconStyle: box
      boxForm: horizontal
      position: bottomCenter
      networks: Weibo,Wechat,Douban,QQZone,Twitter,Facebook
      #左下角悬浮按钮
  float:
    enable: true
    options:
      iconStyle: box
      boxForm: horizontal
      position: middleRight
      networks: Weibo,Wechat,Douban,QQZone,Twitter,Facebook
```

## 访问次数

```yaml
# busuanzi统计
busuanzi_count:
  enable: true
  # 总访客数
  total_visitors: true
  total_visitors_icon: user
  # 总浏览量
  total_views: true
  total_views_icon: eye
  # 文章浏览量
  post_views: true
  post_views_icon: eye
```

## 顶部阅读进度条

```yaml
reading_progress:
  enable: true
  color: "#37c6c0"
  height: 2px
```

## 加载动画

```yaml
motion:
# 启用
  enable: true
  # 异步加载
  async: true
  transition:
    # Transition variants:
    # fadeIn | fadeOut | flipXIn | flipXOut | flipYIn | flipYOut | flipBounceXIn | flipBounceXOut | flipBounceYIn | flipBounceYOut
    # swoopIn | swoopOut | whirlIn | whirlOut | shrinkIn | shrinkOut | expandIn | expandOut
    # bounceIn | bounceOut | bounceUpIn | bounceUpOut | bounceDownIn | bounceDownOut | bounceLeftIn | bounceLeftOut | bounceRightIn | bounceRightOut
    # slideUpIn | slideUpOut | slideDownIn | slideDownOut | slideLeftIn | slideLeftOut | slideRightIn | slideRightOut
    # slideUpBigIn | slideUpBigOut | slideDownBigIn | slideDownBigOut | slideLeftBigIn | slideLeftBigOut | slideRightBigIn | slideRightBigOut
    # perspectiveUpIn | perspectiveUpOut | perspectiveDownIn | perspectiveDownOut | perspectiveLeftIn | perspectiveLeftOut | perspectiveRightIn | perspectiveRightOut
	# 文章摘要动画
    post_block: bounceIn
    # 加载各种页面动画（分类，关于，标签等等）
    post_header: fadeIn
    # 文章详情动画
    post_body: fadeIn
    # 
    coll_header: fadeIn
    # Only for Pisces | Gemini.
    # 侧边栏（人物头像的那部分）
    sidebar: fadeIn
```

> 解释待考证

## 搜索功能

`NexT`自带提供了两个搜索

- `algolia_search`
- `local_search`



其实这个`local_search`已经很好用了，配置`algolia_search`挺麻烦的，而且搜索功能也用的不多



毕竟有万能的`Ctrl + F` 

```yaml
local_search:
  enable: true
  # if auto, trigger search by changing input
  # if manual, trigger search by pressing enter key or search button
  trigger: auto
  # show top n results per article, show all results by setting to -1
  top_n_per_article: 1
  # unescape html strings to the readable one
  unescape: false
```

# 添加RSS订阅

```bash
npm install hexo-generator-feed --save
```

```yaml
# Extensions
plugins:
   hexo-generate-feed
feed: # RSS订阅插件
  type: atom
  path: atom.xml
  limit: 0 #0就是代表所有
```

> 这个功能就是一个RSS装饰条而已，一般也没有什么用

# 添加置顶博客

https://blog.csdn.net/qwerty200696/article/details/79010629

# 显示最近博客

```bash
npm install hexo-related-popular-posts --save
```

```yaml
related_posts:
  enable: true
  title: # custom header, leave empty to use the default one
  display_in_home: false
  params:
    maxCount: 5
    #PPMixingRate: 0.0
    #isDate: false
    #isImage: false
    #isExcerpt: false
```



# 点击爱心效果

## 创建js文件

在`/themes/next/source/js/src`下新建文件`clicklove.js`，接着把该[链接](http://7u2ss1.com1.z0.glb.clouddn.com/love.js)下的代码拷贝粘贴到`clicklove.js`文件中。
代码如下：

```js
!function(e,t,a){function n(){c(".heart{width: 10px;height: 10px;position: fixed;background: #f00;transform: rotate(45deg);-webkit-transform: rotate(45deg);-moz-transform: rotate(45deg);}.heart:after,.heart:before{content: '';width: inherit;height: inherit;background: inherit;border-radius: 50%;-webkit-border-radius: 50%;-moz-border-radius: 50%;position: fixed;}.heart:after{top: -5px;}.heart:before{left: -5px;}"),o(),r()}function r(){for(var e=0;e<d.length;e++)d[e].alpha<=0?(t.body.removeChild(d[e].el),d.splice(e,1)):(d[e].y--,d[e].scale+=.004,d[e].alpha-=.013,d[e].el.style.cssText="left:"+d[e].x+"px;top:"+d[e].y+"px;opacity:"+d[e].alpha+";transform:scale("+d[e].scale+","+d[e].scale+") rotate(45deg);background:"+d[e].color+";z-index:99999");requestAnimationFrame(r)}function o(){var t="function"==typeof e.onclick&&e.onclick;e.onclick=function(e){t&&t(),i(e)}}function i(e){var a=t.createElement("div");a.className="heart",d.push({el:a,x:e.clientX-5,y:e.clientY-5,scale:1,alpha:1,color:s()}),t.body.appendChild(a)}function c(e){var a=t.createElement("style");a.type="text/css";try{a.appendChild(t.createTextNode(e))}catch(t){a.styleSheet.cssText=e}t.getElementsByTagName("head")[0].appendChild(a)}function s(){return"rgb("+~~(255*Math.random())+","+~~(255*Math.random())+","+~~(255*Math.random())+")"}var d=[];e.requestAnimationFrame=function(){return e.requestAnimationFrame||e.webkitRequestAnimationFrame||e.mozRequestAnimationFrame||e.oRequestAnimationFrame||e.msRequestAnimationFrame||function(e){setTimeout(e,1e3/60)}}(),n()}(window,document);
```



## 修改_layout.swig

在`\themes\next\layout\_layout.swig`文件末尾添加：

```
<!-- 页面点击小红心 -->
<script type="text/javascript" src="/js/src/clicklove.js"></script>
```

# 添加文章字数统计

安装依赖

```bash
npm install hexo-symbols-count-time --save
```

配置**主题**的`_config.yaml`

```yaml
# Post wordcount display settings
# Dependencies: https://github.com/theme-next/hexo-symbols-count-time
symbols_count_time:
  separated_meta: true
  separated_meta: true
  item_text_post: true
  item_text_total: true
  awl: 4 # 平均字符长度，这是英文的平均长度
  wpm: 275 #平均阅读速度
  
```



**但~~~~是！**

这个功能是集成在`NexT`里面的，但是我没运行成功

查看了下官方的文档https://github.com/theme-next/hexo-symbols-count-time

才发现需要在**网站**的`_config.yaml`也要配置

`D:\MyBlog\_config.yml`

```yaml
symbols_count_time:
  symbols: true  #启用符号
  time: true    #启用阅读时间
  total_symbols: true    #启用符号，总站文章
  total_time: true      #启用时间，总站文章
```



# 搜索功能Algolia_search

```
# Algolia Search
# See: https://github.com/theme-next/hexo-theme-next/blob/master/docs/ALGOLIA-SEARCH.md
# Dependencies: https://github.com/theme-next/theme-next-algolia-instant-search
```



# Note的使用

之前看到过别人使用“有颜色的引用”，着实好奇，然后google了好久也没看到怎么个用法

原来是用了Note！

## 配置

打开主题的`_config.yml`

修改为以下配置

```yml
note:
  style: flat
  icons: false
  border_radius: 3
  light_bg_offset: 0
```

其实就是启用note而已

## 风格

可以选择几种风格

- `simple` 
- `modern` 
- `flat`
- `disable`

![1547663317610](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/hexonote/1547663317610.png)

![1547663494704](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/hexonote/1547663494704.png)

 ![1547663551300](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/hexonote/1547663551300.png)

![1547663610589](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/hexonote/1547663610589.png)

(哈哈，`disabled`就是没有)



对了，如果设置`icons`为`true`的话，就是会下面这个样子

![1547663053078](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/hexonote/1547663053078.png)

## 使用方法

**直接在**`markdown`文章里面写上

```js
{% note class_name %}现在该说点什么呢{% endnote %}
```

其中，`class_name` 可以是以下列表中的一个值：

- `default`
- `primary`
- `success`
- `info`
- `warning`
- `danger`

![1547663961713](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/hexonote/1547663961713.png)

(图片来自官网介绍)

# 文本居中的引用

**直接在**`markdown`中写

```html
<!-- HTML方式: 直接在 Markdown 文件中编写 HTML 来调用 -->
<!-- 其中 class="blockquote-center" 是必须的 -->

<blockquote class="blockquote-center">blah blah blah</blockquote>

<!-- 标签 方式，要求版本在0.4.5或以上 -->
{% centerquote %}blah blah blah{% endcenterquote %}
<!-- 标签别名 -->
{% cq %} blah blah blah {% endcq %}
```

比如我写上

```html
{% cq %} 鲁迅说：我不用Hexo写博客 {% endcq %}
```

效果是

![1547664665317](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/hexonote/1547664665317.png)

# 自定义文章摘要

其实设置这个的目的只是因为我之前的首页文章摘要没有格式，如下图

![1547664903317](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/hexonote/1547664903317.png)

**设置手动摘要**



在需要的开始正文的地方添加

```html
<!-- more --> 
```

**源代码**

（是用`typora`查看的）

![1547d65029443](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/hexonote/1547665029443.png)

**效果**

![1547665194832](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/hexonote/1547665194832.png)



是不是好看多了！



**其实可以设置成自动生成摘要**

 那样的话如果插入了标签，它也能识别的

 不然之前我没手动设置摘要的文章默认会在首页显示整篇文章，太恐怖了

 所以可以在每篇文章最后添加<!-- more --> ，就会根据自动设置的长度截取下来保存格式显示摘要

> 修改模板文件
>
> blog\scaffolds\post.md
>
> 最后添加<!-- more -->



但是这个标签似乎对`markdown`不能很好支持，如果有大量`markdown`语法嵌入进去，还是用引用吧



# 主题颜色及大小定制

修改文件`D:\MyBlog\themes\next\source\css\_variables\custom.styl`

## 添加自己喜欢的颜色

在这个网站：https://htmlcolorcodes.com/zh/

找到自己喜欢的颜色的十六进制数

```stylus

//蓝绿色
$Mycolor  = #17A589  
$Light_Mycolor = #76D7C4
$Medium_Mycolor = #48C9B0

// segment fault同款绿
$MySecondColor = #009a61
//$Light_MySecondColor = #7DCEA0
$Dark_MySecondColor = #006741

// segment fault同款绿
$MyThirdColor = #009a61
//$Light_MyThirdColor = #27AE60

//紫色
$MyCodeColor =  #8A2BE2 
```

## 主题颜色

```stylus
//主题颜色
$body-bg-color                = $MyThirdColor
```

## 链接颜色

```stylus
//超链接颜色
$link-color                   = $MySecondColor
$link-hover-color             = $Dark_MySecondColor
//超链接下面的横线
$link-decoration-color        = $MySecondColor
$link-decoration-hover-color  = $Dark_MySecondColor
```

## 顶部条颜色

```stylus
//顶部条的颜色
$headband-height                = 2px
$headband-bg                    = $MyThirdColor
```

## 字体大小

```stylus
//文章内标题字体大小
$font-size-headings-step    = 6px
$font-size-headings-base    = 32px
//基本字体大小
$font-size-base           = 15px
```

## 文章标题大小和颜色

`D:\MyBlog\themes\next\source\css\_common\components\post\post-title.styl`

```stylus
.posts-expand .post-title {
  word-wrap();
  text-align: center;
  //下面这两个要修改去D:\MyBlog\themes\next\source\css\_variables\base.styl
  //字体粗细
  font-weight: $posts-expand-title-font-weight;
  //字体大小
  font-size: $posts-expand-title-font-size
  //修改为主题色
  color: $MySecondColor;

  if hexo-config('post_edit.enable') {
    .post-edit-link {
      color: #bbb;
      display: inline-block;
      float: right;
      border-bottom: none;
      the-transition-ease-in();
      margin-left: -1.2em;
      +mobile-small() {
        margin-left: initial;
      }
      &:hover {
        color: black;
      }
    }
  }
}

.posts-expand .post-title-link {
  display: inline-block;
  position: relative;
  color: $black-light;
  border-bottom: none;
  //鼠标悬浮式下划线距离标题的距离
  line-height: 1.8;
  vertical-align: top;

  &::before {
    content: "";
    position: absolute;
    width: 100%;
    height: 2px;
    bottom: 0;
    left: 0;
    background-color: #000;
    visibility: hidden;
    transform: scaleX(0);
    the-transition();
  }

  &:hover::before {
    visibility: visible;
    transform: scaleX(1);
  }
  
  &:hover{
      //鼠标悬浮时标题设置为自己的主题色
	color:$MyThirdColor;
  }

  .fa {
    font-size: 20px;
    margin-left: 5px;
  }
}

```



## 侧边栏颜色

```stylus
//目录颜色
$sidebar-nav-hover-color          = $Mycolor
$sidebar-highlight                = $Mycolor
//日志，分类，标签之间的分割线
$site-state-item-border-color     = $Mycolor
//目录鼠标悬浮颜色
$toc-link-hover-color                 = $Light_Mycolor
$toc-link-hover-border-color          = $Light_Mycolor
```

## 按钮

首页的”查看全文“等按钮

```stylus
//按钮
$btn-default-radius           = 2px
$btn-default-bg               = white
$btn-default-color            = $text-color
$btn-default-border-color     = $text-color
$btn-default-hover-color      = white
$btn-default-hover-bg         = $MyThirdColor
```



## 返回顶部按钮

```stylus
//返回顶部按钮
$b2t-sidebar-bg-color         = $Medium_Mycolor
```

## 代码颜色

```stylus
//代码
$code-font-family               = $font-family-monospace
$code-font-size                 = 14px
$code-font-size                 = unit(hexo-config('font.codes.size'), px) if hexo-config('font.codes.size') is a 'unit'
$code-border-radius             = 3px
$code-foreground                = $MyCodeColor
$code-background                = $gainsboro
```

## 主页文章添加边框阴影效果

打开 `themes/next/source/css/_custom/` 下的 `custom.styl`，向里面加代码：

```stylus
.post {
   margin-top: 0px;
   margin-bottom: 60px;
   padding: 25px;
   -webkit-box-shadow: 0 0 5px rgba(202, 203, 203, .5);
   -moz-box-shadow: 0 0 5px rgba(202, 203, 204, .5);
}
```

## 修改文章间分割线

打开 `themes/next/source/css/_common/components/post/` 下的 `post-eof.styl` ，修改：

```stylus
.posts-expand {
  .post-eof {
    display: block;
    width: 0%;
    height: 0px;
    background: $grey-light;
    text-align: center;
  }
}
```

## 更改右上角GihHub角标

有的东西不能在自定义的样式文件中更改

修改文件`D:\MyBlog\themes\next\layout\_partials\github-banner.swig`

把它的颜色`github_banner_bg_color`和`github_banner_fg_color`设置成主题一样的颜色

```stylus
{% if theme.github_banner %}
  {% set github_banner_bg_color = '#1E8449 ' %}
  {% set github_banner_fg_color = '#fff' %}
  {% set github_URL = theme.github_banner.split('||')[0] | trim %}
  {% set github_title = theme.github_banner.split('||')[1] | trim %}

  {% set github_image = '<svg width="80" height="80" viewBox="0 0 250 250" style="fill: ' + github_banner_bg_color + '; color: ' + github_banner_fg_color + '; position: absolute; top: 0; border: 0; right: 0;" aria-hidden="true"><path d="M0,0 L115,115 L130,115 L142,142 L250,250 L250,0 Z"></path><path d="M128.3,109.0 C113.8,99.7 119.0,89.6 119.0,89.6 C122.0,82.7 120.5,78.6 120.5,78.6 C119.2,72.0 123.4,76.3 123.4,76.3 C127.3,80.9 125.5,87.3 125.5,87.3 C122.9,97.6 130.6,101.9 134.4,103.2" fill="currentColor" style="transform-origin: 130px 106px;" class="octo-arm"></path><path d="M115.0,115.0 C114.9,115.1 118.7,116.5 119.8,115.4 L133.7,101.6 C136.9,99.2 139.9,98.4 142.2,98.6 C133.8,88.0 127.5,74.4 143.8,58.0 C148.5,53.4 154.0,51.2 159.7,51.0 C160.3,49.4 163.2,43.6 171.4,40.1 C171.4,40.1 176.1,42.5 178.8,56.2 C183.1,58.6 187.2,61.8 190.9,65.4 C194.5,69.0 197.7,73.2 200.1,77.6 C213.8,80.2 216.3,84.9 216.3,84.9 C212.7,93.1 206.9,96.0 205.4,96.6 C205.1,102.4 203.0,107.8 198.3,112.5 C181.9,128.9 168.3,122.5 157.7,114.1 C157.9,116.9 156.7,120.9 152.7,124.9 L141.0,136.5 C139.8,137.7 141.6,141.9 141.8,141.8 Z" fill="currentColor" class="octo-body"></path></svg>' %}

  {{ next_url(github_URL, github_image, {class: 'github-corner', title: github_title, "aria-label": github_title}) }}
{% endif %}

```

如果需要平板和手机的视图也要显示一致的话，还要更改`D:\MyBlog\themes\next\source\css\_common\components\header\github-banner.styl`

```css
.github-corner {
  scheme = hexo-config('scheme');
  //bg_color = unquote(hexo-config('android_chrome_color'));
  bg_color = #1E8449 
  mobile_layout_economy = hexo-config('mobile_layout_economy');

  :hover .octo-arm {
    animation: octocat-wave 560ms ease-in-out;
  }
  @keyframes octocat-wave {
    0%,100% {
      transform: rotate(0);
    }
    20%,60% {
      transform: rotate(-25deg);
    }
    40%,80% {
      transform: rotate(10deg);
    }
  }
  +tablet-mobile() {
    >svg {
      if (scheme == 'Pisces') || (scheme == 'Gemini') {
        fill: #fff !important;
        color: bg_color !important;
      }
    }
    .github-corner:hover .octo-arm {
      animation: none;
    }
    .github-corner .octo-arm {
      animation: octocat-wave 560ms ease-in-out;
    }
  }
  +mobile() {
    >svg {
      if (scheme == 'Mist') {
        top: inherit !important;
        margin-top: -50px;
        if mobile_layout_economy {
          +mobile-small() {
            margin-top: initial;
          }
        }
      }
    }
  }
}

```

# 加密文章

https://github.com/MikeCoder/hexo-blog-encrypt/blob/master/ReadMe.zh.md

亲测纯数字密码有问题！！！还是设置字母密码吧！



# 网站速度优化

安装gulp

```bash
$ npm install gulp -g
```

安装插件

```bash
$ npm install gulp-minify-css gulp-uglify gulp-htmlmin gulp-imagemin gulp-htmlclean  gulp --save
```



# SEO优化

## 百度主动提交

## 百度自动提交

## 百度Sitemap提交

## 谷歌Sitemap提交



## 小插曲

 发现了红色项 ![1547679295960](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/linuxnote/1547679295960.png) 

应该是百度的什么东西 在本地的博客文件夹里搜索不到这个 `push.js` 文件

 ![1547679374656](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/linuxnote/1547679374656.png)

 右键复制地址后，发现不是来自我的网站，而是 https://zz.bdstatic.com/linksubmit/push.js 但是点击不进去 搜索之后发现这是百度的域名，而且在做的好像是爬取数据（老本行） 但是为什么没加载出来呢？

 **因为被我的 chrome 拦截掉了哈哈哈哈! ** 

右键 `AdBlock`，在此网站暂停就好了 （怪不得我的网站收录这么差） 

![1547679564937](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/linuxnote/1547679564937.png)

# 死亡警告

千万不要在hexo server的时候更改博客内容，不然内容会被**清空**的！

# 参考资料

[自动更换背景图片](https://ihaoming.top/archives/d0564105.html)