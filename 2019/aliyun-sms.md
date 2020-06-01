---
title: 使用阿里云短信推送服务发送验证码
urlname: aliyun-sms
tags:
  - Service
  - AliYun
  - JAVA
copyright: true
category: Service
date: 2019-05-16 14:35:41
---

## 前言

最近课程项目做一个网站，需要实现注册、忘记密码等功能的推送验证码的功能

而且根据相关法律法规，用户也需要手机号认证......（反正很多网站都是这么说的)

<!-- more --> 

## 介绍

本来想弄一个免费的短信服务商，<http://www.mob.com/product/sms>这家就挺不错的，支持还挺全，有

- IOS
- Android
- Unity3d
- Cocos2d
- flutter
- apicloud

**而且都是免费的！**

但是没有`web`端

客服回答说要有APP，要上线，要充值，每条五分......瞬间没了好感

> 而且不管使用什么SDK都要实名认证，身份证照片啊，手持身份证照片啊，说实话把这些信息给这些小企业有点信不过

![1557992042904](https://blog.janking.cn/post/aliyun-sms/1557992042904.png)

开始准备看看他们提供的`Android`源码，你不也是调用`API`去发送短信吗，那我就把这个接口给 ~~偷过来~~ 窃过来不就可以了

当我一层一层地点开各种方法时，看到的都是类似下图这种`a`啊，`b`啊的方法名，还有参数也不明白意思，看来做了混淆防止反编译，算了，我这点渣渣水平就不去像这种骚操作了。

> 如果真的不想花钱的话，可以找一台手机（安卓或IOS）使用它们的SDK，然后从网站上去唤醒这个手机发送验证码的功能，间接使用，当然速度会很慢啊，手机一直联网可能会断网啊，增加开发手机端的代码量啊很多缺点，但是不要钱也是很香的

![1557992497442](https://blog.janking.cn/post/aliyun-sms/1557992497442.png)

还是决定老老实实用付费的吧

这个选择就多了去了，普遍是`0.05`元一条，有的小网站可以做到`0.03`元左右，还是用大服务商的吧，稳定信得过，什么阿里云，腾讯云，百度云，网易云啥的都有

这里我选择的是**阿里云**

> 顺便说一下，良心的亚马逊云AWS也有免费的短信包，之前我就一直在用这个测试，但是后来才发现只有美国的手机号码才免费，于是花了我0.1美元......



## 资费

定价如下，可以选择套餐包或者预付费，不过套餐是`5000`条起卖`225`元（emmmm对于我们这种蚊子型网站有点浪费啊）所以还是选择预付费吧，用多少付多少

![1557993109520](https://blog.janking.cn/post/aliyun-sms/1557993109520.png)

而且新开通的用户还会`送100条短信`，估计是考虑到测试的需要

![1557994534479](https://blog.janking.cn/post/aliyun-sms/1557994534479.png)

[开通链接](<https://www.aliyun.com/product/sms?spm=a2c4g.11186623.cloudEssentials.54.32ae18b5VwaZpN>)

然后需要两个基本的东西，一个是**签名**，一个是**模板**

如下

- 签名就是【】框里的东西
- 模板就是【】框后面的内容，可以有变量

![1557995229495](https://blog.janking.cn/post/aliyun-sms/1557995229495.png)

## 添加签名

然后进入**控制台->短信服务->国内消息->添加签名**，可以看到我已经添加了一条签名

![1557994815227](https://blog.janking.cn/post/aliyun-sms/1557994815227.png)

签名填写**服务的名称**，场景选择**验证码**（通用类型需要验证很多东西），申请说明是给审核人员看的，一般两小时会完成审核

![1557994962200](https://blog.janking.cn/post/aliyun-sms/1557994962200.png)

## 添加模板

选择**模板管理->添加模板**

![1557995519506](https://blog.janking.cn/post/aliyun-sms/1557995519506.png)

选择验证码

输入模板名称（这个名称是给自己看的不会出现在短信中）

输入模板内容（可以有变量，用${}包起来，发短信的时候可以动态控制）

还有申请说明

![1557995630789](https://blog.janking.cn/post/aliyun-sms/1557995630789.png)

## 测试发送

**当签名和模板都审核通过后**

点击快速学习，使用给定的签名和模板进行测试，填入变量和手机号码，点击发送短息就可以测试发送消息

另外这个查看API Demo会提供各种代码调用API的示例，方便整合进自己的服务中，后面还会用到

![1557996518909](https://blog.janking.cn/post/aliyun-sms/1557996518909.png)



## 添加访问密钥

使用代码调用API时，要通过`accessKeyId`和`secret`验证身份

点击右上角个人头像，选择**访问控制**

> 为什么是点访问控制而不是`accesskeys`呢？
>
> - 因为`accesskeys`里面也会建议你使用子用户进行访问。
>
> 那为什么要子用户进行访问呢？
>
> - 比如在阿里云开通了很多付费服务都需要`accesskey`验证，如果使用主账户的`accesskey`不小心泄露了，那么所有的付费服务都被泄露了，可能带来很大的财产损失和信息丢失
> - 如果使用子用户就可以动态控制每个用户的权限，比如a用户只有发短息的权限，再不济泄露了也不会影响其他的服务，将损失降为最小

![1557996822905](https://blog.janking.cn/post/aliyun-sms/1557996822905.png)

选择**用户->新建用户**，可以看到我已经建立了两个用户分别用于短信和邮件服务

![1557997147010](https://blog.janking.cn/post/aliyun-sms/1557997147010.png)

写好**登录名称，显示名称，勾选编程访问**就好了，如果是控制台登录的话其实就相当于一个阿里云账户可以在网站上登录，这个暂时不需要

![1557997237018](https://blog.janking.cn/post/aliyun-sms/1557997237018.png)

下图中的`AccessKeyID`和`AccessKeySceret`就是编程需要的参数了

> 不要试图使用我的这个`AccessKey`干坏事哦，因为 ~~我账户里没钱~~ 它已经被删掉啦

需要注意的是账号一旦创建完成，就**再也看不到它们了**，所以要提前点击**复制**或者**下载CSV文件**进行备份

然后再点击**添加权限**

![1557997601670](https://blog.janking.cn/post/aliyun-sms/1557997601670.png)

对于短信推送

输入`sms`进行搜索，选择`AliyunDysmsFullAccess` ，点击确定

![1557997793072](https://blog.janking.cn/post/aliyun-sms/1557997793072.png)

## 使用代码调用短信服务

回到**控制台->短信服务->快速学习**，点击查看`API DEMO`

![1557998341688](https://blog.janking.cn/post/aliyun-sms/1557998341688.png)

在右侧选择需要的语言，将代码全部复制下来

![1557998393213](https://blog.janking.cn/post/aliyun-sms/1557998393213.png)

### 添加依赖

以`JAVA`为例

因为使用的`api`是阿里云自家的，所以官方默认`jar`肯定是没有

以`maven`为例

在`dependencies`中添加依赖，这个依赖在API DEMO中的注释部分给了出来，如

```
<dependency>
    <groupId>com.aliyun</groupId>
    <artifactId>aliyun-java-sdk-core</artifactId>
    <version>4.1.0</version>
</dependency>
```

![1557998592992](https://blog.janking.cn/post/aliyun-sms/1557998592992.png)

### 完整代码

```java
//填入有短信推送服务权限的AccessKey id和secret
DefaultProfile profile = DefaultProfile.getProfile("cn-hangzhou", "<accessKeyId>", "<accessSecret>");
IAcsClient client = new DefaultAcsClient(profile);
CommonRequest request = new CommonRequest();
//设置https调用
//request.setProtocol(ProtocolType.HTTPS);
request.setMethod(MethodType.POST);
request.setDomain("dysmsapi.aliyuncs.com");
request.setVersion("2017-05-25");
request.setAction("SendSms");
request.putQueryParameter("RegionId", "cn-hangzhou");
//设置收信手机号码
request.putQueryParameter("PhoneNumbers", "12233334444");
//之前申请的签名名称
request.putQueryParameter("SignName", "TimeIsMoney");
//之前申请的模板CODE
request.putQueryParameter("TemplateCode", "SMS_165380907");
//模板里面的变量赋值，使用JSON格式
request.putQueryParameter("TemplateParam", "{\"code\":\"6666\"}");
try {
    CommonResponse response = client.getCommonResponse(request);
    System.out.println(response.getData());
} catch (ServerException e) {
    e.printStackTrace();
} catch (ClientException e) {
    e.printStackTrace();
}
```

## 密钥安全保障

如果是开源项目的话，这样子会直接把密钥暴露在互联网中，相当不安全，所以需要做一点措施，把密钥和代码分开

在`resources`包里添加包`privateKey`

![1557999887278](https://blog.janking.cn/post/aliyun-sms/1557999887278.png)

在这个包里新建`SMSKey.txt`存储密钥

写上两行数据，一行是`ID`一行是`secret`

![1558000026585](https://blog.janking.cn/post/aliyun-sms/1558000026585.png)

修改代码，添加两个私有变量`accessKeyId`和`secret`，并且在静态代码块里把它们赋值为`txt`中的值

```java
private static final String accessKeyId;
private static final String secret;
/**
 * 初始化获取私钥
 * */
static {
    ClassPathResource smsKey = new ClassPathResource("privateKey/SMSKey.txt");
    System.out.println(smsKey);
    Scanner scanner = null;
    try {
        scanner = new Scanner(smsKey.getInputStream());
    } catch (IOException e) {
        e.printStackTrace();
    }
    accessKeyId = scanner.nextLine();
    secret = scanner.nextLine();
    scanner.close();
}
```

然后调用的时候使用这两个变量就好了

```java
DefaultProfile profile = DefaultProfile.getProfile("cn-hangzhou", accessKeyId, secret);
```

> 邮件的密钥同理

**还有一步，不能把这两个`txt`文件也传到`git`仓库中去**

在`gitignore`中添加

```.gitignore
EmailKey.txt
SMSKey.txt
```

现在就可以安心地把代码`push`上去了。

另外这只是开源代码遇到的问题，如果真的部署到了服务器，还会有以下问题值得思考

- 获取验证码的网站页面，没有做防止非法获取验证码的措施，比如最简单的图形校验码。（常见于网站上获取验证码）

  **建议**：点击获取验证码之前做一个校验，比如需要正确输入图形验证码、拖动验证等

- 发送短信的接口暴露，且没有做加密措施，遭受恶意调用。（此类常见于APP侧获取验证码）

  **建议**：在短信接口做一些加密措施（例如加入图形验证等方式），防止非法调用。

## 效果

![img](https://blog.janking.cn/post/aliyun-sms/B89C2157ADE4FA626CAF22172C2F4F82.jpg)

