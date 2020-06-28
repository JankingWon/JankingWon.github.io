---
url: miniprogram-npm-error
data: 2020-06-11 18:07
---



# 正确使用微信小程序组件库，解决报错Component is not found in path

本人node小白，在开发微信小程序的时候想偷懒使用一个组件库，可是一直报这个错误

```
Component is not found in path "@vant/weapp/dist/image/index"
```

太焦虑了，找了好久总结出正确的使用步骤，如果也不会的小伙伴按照下列步骤做就不会出问题啦！

## 正确使用步骤

以小程序组件库 https://github.com/youzan/vant-weapp 为例

1. 在小程序项目**根目录**打开命令行，输入

   ```bash
   npm i @vant/weapp -S --production
   ```

2. 然后你会发现项目内多了一个「node_modules」目录和「package-lock.json」文件

   ![](https://cdn.jsdelivr.net/gh/jankingwon/jankingwon.github.io@latest/2020/image/20200611173950.png)

3. 在小程序根目录输入 `npm init`初始化项目

   ![](https://cdn.jsdelivr.net/gh/jankingwon/jankingwon.github.io@latest/2020/image/20200611175145.png)

4. 在生成的 「package.json」文件中的「dependencies」中填入你需要依赖的模块和版本号，如

   ![](https://cdn.jsdelivr.net/gh/jankingwon/jankingwon.github.io@latest/2020/image/20200611175549.png)

5. 打开「详情」，勾上「使用npm模块」

   ![](https://cdn.jsdelivr.net/gh/jankingwon/jankingwon.github.io@latest/2020/image/20200611174046.png)

6. 点击「工具」-> 「构建npm」 

   ![](https://cdn.jsdelivr.net/gh/jankingwon/jankingwon.github.io@latest/2020/image/20200611174220.png)

7. 就成功啦！

   ![](https://cdn.jsdelivr.net/gh/jankingwon/jankingwon.github.io@latest/2020/image/20200611180048.png)

   此时你的项目根目录会多一个「miniprogram_npm」的文件夹

   ![](https://cdn.jsdelivr.net/gh/jankingwon/jankingwon.github.io@latest/2020/image/20200611180156.png)

8. 不过使用起来还要注意，按照如下的方式引用，而不是使用路径引用哦

   ![](https://cdn.jsdelivr.net/gh/jankingwon/jankingwon.github.io@latest/2020/image/20200611180252.png)

9. 祝没有bug！

