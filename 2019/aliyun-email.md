---
title: 使用阿里云邮件推送服务发送验证码
urlname: aliyun-email
tags:
  - AliYun
  - Java
  - Service
copyright: true
category: Service
date: 2019-05-16 20:34:55
---

## 介绍

其实邮件推送就比较简单了，其实可以使用自己的邮件账号在代码中使用`SMTP`、`POP3`或者`IMAP`登陆，其实本来也是打算这么干的，因为很多邮件推送服务也是要收费的，但是呢~~阿里云（又是阿里云）有个活动

<!-- more --> 

> 放这个图着实有点打广告的嫌疑，但是站在个人角度确实挺优惠

![1558006002072](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-email/1558006002072.png)

因为使用大企业已经成熟的邮件推送服务，稳定性肯定不用担心，而且听说遇到问题还可以咨询技术顾问，今天我就收到了阿里云客服的电话问使用情况并告知有技术问题可以点击右上角咨询，服务还是蛮棒的！

这个活动每天可以免费发`200`封邮件，参考套餐包的价格是`50000`封`81`元，哪有用户一天到晚注册，一天到晚忘记密码，而且要是真的有这么多用户再考虑付费或者自己实现邮件推送功能吧（哪有稳定什么的区别，主要是怕麻烦，能用就行......

## 添加域名

要注意的是，用来发送邮件推送的邮箱域名必须是自己拥有的或者能控制`DNS`解析的

试想如果一个用户收到123456789@qq.com发来的验证码，那该有多害怕啊......

国内也有很多域名提供商，比如**腾讯云**（新网的代理），**阿里云**（旗下品牌万网），但是要备案

推荐使用[https://www.namesilo.com](https://www.namesilo.com/)进行域名购买，原因是

- 国外的域名提供商**不用备案！**
- **价格最便宜**，同样域名比国内域名商都要便宜！（~~所以网站UI比较差，这不关键~~）
- 支持**支付宝**

> 这里得声明下，备案的政策还是好的，可以防止一些非法网站的存在，只是我们这种超级小型网站就没有必要把时间花在这个上面，之前我在腾讯云也备案过几个域名，大概周期20多天吧，还要身份认证、幕布拍照什么的，当然如果运营的好的比较稳定的网站，背个案还是很关键的，比如我的[博客网站](https://blog.janking.cn)，嘿嘿嘿

注册域名过程就不说了，注册完成之后如果觉得DNS解析比较慢或者不稳定~~或者网站UI看不下去~~可以 把`DNS`解析到[CloudFlare](https://www.cloudflare.com/)

添加下面两行记录解析到`CloudFlare`，以后就去`CloudFlare`管理`DNS`记录了，一时就是把`namesilo`当卖域名的，后来除了续费就不用进这个网站了

```
fred.ns.cloudflare.com
robin.ns.cloudflare.com
```

> 当然也可以不设置DNS解析到其它站点，`namesilo`就可以，但是生效比较慢，操作方法为点击下图中的`DNS Records`

![1558007980809](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-email/1558007980809.png)

现在来修改`DNS`到阿里云

在**邮件推送控制台->发件域名->新建域名->输入域名**

我这里使用的是`service.域名`

> 它提醒不要用企业邮箱的域名，可能会导致企业邮箱收件异常

![1558008506058](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-email/1558008506058.png)

然后点击**配置**，它会提示你需要添加哪些`DNS`记录，必要的这三条就好了，第四条跟踪邮件需要备案

如果不会配置`DNS`，可以点击它的**示例**

![1558008576027](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-email/1558008576027.png)

我在`CloudFlare`的配置如下

![1558008867957](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-email/1558008867957.png)

![1558008975261](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-email/1558008975261.png)

![1558009112933](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-email/1558009112933.png)

这是第四条记录，非必要，有黄色云朵表示开启`CDN`，因为在海外其实会比较慢，推荐关闭

![1558009198865](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-email/1558009198865.png)

等一会儿，手动点击发信域名的**验证**，如果状态变为

**可使用-未备案** 或 **可使用-未验证CName** 或 **验证通过**表示可以发送邮件了

![1558009803023](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-email/1558009803023.png)

## 添加发件地址

回到**邮件推送控制台->发信地址->新建发信地址**

选择发信域名，账号，发信类型

> 回信地址可以不写
>
> 官方说
>
> “触发类邮件指注册激活、密码找回等；批量邮件指营销推广、订阅期刊等。不同类型邮件的发送限制不同，请根据邮件类型选择。”
>
> 其实在发送的时候都是一样的，我还以为触发邮件可以自定义触发的条件呢，找了半天也没找到。。。。。。

![1558009873331](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-email/1558009873331.png)

如果邮件的状态变为**正常**后，表示可以发件了

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

![1557996822905](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-email/1557996822905.png)

选择**用户->新建用户**，可以看到我已经建立了两个用户分别用于短信和邮件服务

![1557997147010](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-email/1557997147010.png)

写好**登录名称，显示名称，勾选编程访问**就好了，如果是控制台登录的话其实就相当于一个阿里云账户可以在网站上登录，这个暂时不需要

![1557997237018](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-email/1557997237018.png)

下图中的`AccessKeyID`和`AccessKeySceret`就是编程需要的参数了

> 不要试图使用我的这个`AccessKey`干坏事哦，因为 ~~我账户里没钱~~ 它已经被删掉啦

需要注意的是账号一旦创建完成，就**再也看不到它们了**，所以要提前点击**复制**或者**下载CSV文件**进行备份

然后再点击**添加权限**

![1557997601670](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-email/1557997601670.png)

对于邮件推送

输入`mail`进行搜索，选择`AliyunDirectMailAccess` ，点击确定

![1557997917185](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-email/1557997917185.png)



## 使用代码调用邮件服务

有两种调用邮件推送的方式

一种是使用阿里云给的API，只需要提供必要的参数；

另一种是使用`SMTP`就相当于阿里云提供了一个发件服务器，然后你使用代码登录进去发邮件，我这里选择第一种

### 添加依赖

以`JAVA`的`maven`为例

先要添加一个`maven`仓库

```
<repositories>
     <repository>
             <id>sonatype-nexus-staging</id>
             <name>Sonatype Nexus Staging</name>
             <url>https://oss.sonatype.org/service/local/staging/deploy/maven2/</url>
             <releases>
                     <enabled>true</enabled>
             </releases>
             <snapshots>
                     <enabled>true</enabled>
             </snapshots>
     </repository>
</repositories>
```

然后需要添加两个依赖

> 如果短信推送服务已经添加了一个，这里就只需要添加一个

```
<dependency>
    <groupId>com.aliyun</groupId>
    <artifactId>aliyun-java-sdk-core</artifactId>
    <version>4.1.0</version>
</dependency>
<dependency>
    <groupId>com.aliyun</groupId>
    <artifactId>aliyun-java-sdk-dm</artifactId>
    <version>3.1.0</version>
</dependency>
```

![1558011473732](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-email/1558011473732.png)

### 完整代码

```java
import com.aliyuncs.DefaultAcsClient;
import com.aliyuncs.IAcsClient;
import com.aliyuncs.dm.model.v20151123.SingleSendMailRequest;
import com.aliyuncs.dm.model.v20151123.SingleSendMailResponse;
import com.aliyuncs.exceptions.ClientException;
import com.aliyuncs.exceptions.ServerException;
import com.aliyuncs.profile.DefaultProfile;
import com.aliyuncs.profile.IClientProfile;
//import com.aliyuncs.http.MethodType;
public void sample() {        
        // 如果是除杭州region外的其它region（如新加坡、澳洲Region），需要将下面的"cn-hangzhou"替换为"ap-southeast-1"、或"ap-southeast-2"。
    //下面填写密钥
    IClientProfile profile = DefaultProfile.getProfile("cn-hangzhou", "<your accessKey>", "<your accessSecret>");
    IAcsClient client = new DefaultAcsClient(profile);
    SingleSendMailRequest request = new SingleSendMailRequest();
    //使用https加密连接
    request.setProtocol(com.aliyuncs.http.ProtocolType.HTTPS);
    try {
     //request.setVersion("2017-06-22");// 如果是除杭州region外的其它region（如新加坡region）,必须指定为2017-06-22
        request.setAccountName("控制台创建的发信地址");
        request.setFromAlias("发信人昵称");
        request.setAddressType(1);
        //可以不需要
        //request.setTagName("控制台创建的标签");
        //是否需要回信功能
        //request.setReplyToAddress(true);
        request.setToAddress("目标地址");
        //可以给多个收件人发送邮件，收件人之间用逗号分开，批量发信建议使用BatchSendMailRequest方式
        //request.setToAddress("邮箱1,邮箱2");
        request.setSubject("邮件主题");
        request.setHtmlBody("邮件正文");
        SingleSendMailResponse httpResponse = client.getAcsResponse(request);
    } catch (ServerException e) {
        e.printStackTrace();
    }
    catch (ClientException e) {
        e.printStackTrace();
    }
}
```
### 不要进入垃圾箱

可以看到，邮件推送服务没有使用模板，所有内容都是在`setHtmlBody`中传递，我测试了一下，收到的邮件成功地进入了垃圾箱

![1558012047011](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-email/1558012047011.png)

邮件为什么会进入垃圾箱呢？

知乎有很专业的回答<https://www.zhihu.com/question/19574247>

对于我们这种情况，最大的原因是

- **邮件内容格式不友好**，邮件服务器不喜欢没有格式的邮件，最好是`html`格式
- **邮件没有退订按钮**（这个也不算主要原因，第一点改完基本就可以了）

那么怎么发送`html`邮件呢，在 `request.setHtmlBody("邮件正文");`小小的括号里写上一个`html`文件未免太不友好，这就需要我们手动创建邮件模板了

### 使用html模板发送邮件

在`resources/static`里写上两个`html`文件作为邮件的模板，一个用于用户注册，一个用于用户找回密码

![1558012323081](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-email/1558012323081.png)

要注意不要写死内容，要留一些变量

![1558012390942](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-email/1558012390942.png)

那么怎么使用它呢?

新建两个静态常量数组，用来保存**邮件模板的地址**和**邮件的标题**

```java
private static final String[] EmailTemplate = {"static/mail_register_template.html", "static/mail_forget_password_template.html"};
private static final String[] EmailTitle = {"【TIM挣闲钱】注册邮箱验证","【TIM挣闲钱】找回密码邮箱验证"};
```

对于邮件的标题，可以很简单的调用（`type`控制是哪种类型邮件）

```java
request.setSubject(EmailTitle[type]);
```

对于邮件内容，需要读入`html`文件，下面代码最后的`htmlBody`就是模板完整的内容了

> 这里之前我没有用`scanner.nextLine()`而是`scanner.next()`
>
> 最后得到的邮件内容没有换行......没有成功解析为`html`，所以格式极乱

```java
ClassPathResource mailTemplate = new ClassPathResource(EmailTemplate[type]);
System.out.println(mailTemplate);
Scanner scanner = new Scanner(mailTemplate.getInputStream());
String htmlBody = "";
while (scanner.hasNextLine()){
     htmlBody += scanner.nextLine() + System.getProperty("line.separator");
}
```

这样还不够，还要给里面的变量赋值，这就简单了，字符串替换就好了

```java
htmlBody = htmlBody.replace("[address]", address).replace("[code]", code);
request.setHtmlBody(htmlBody);
```

这样就可以很方便地发送通知邮件了，只要几个参数就行了！

## 密钥安全保障

> 这个在[阿里云短信服务](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-sms.html)里已经提到过
>
> 所以直接把内容复制过来了，操作是一样的，就是把邮箱账号的密钥保存到文件里再读取

如果是开源项目的话，这样子会直接把密钥暴露在互联网中，相当不安全，所以需要做一点措施，把密钥和代码分开

在`resources`包里添加包`privateKey`

![1557999887278](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-email/1557999887278.png)

在这个包里新建`SMSKey.txt`存储密钥

写上两行数据，一行是`ID`一行是`secret`

![1558000026585](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-email/1558000026585.png)

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

**还有一步，不能把这个`txt`文件也传到`git`仓库中去**

在`gitignore`中添加

```.gitignore
SMSKey.txt
```

现在就可以安心地把代码`push`上去了。

另外这只是开源代码遇到的问题，如果真的部署到了服务器，还会有以下问题值得思考

- 获取验证码的网站页面，没有做防止非法获取验证码的措施，比如最简单的图形校验码。（常见于网站上获取验证码）

  **建议**：点击获取验证码之前做一个校验，比如需要正确输入图形验证码、拖动验证等

- 发送短信的接口暴露，且没有做加密措施，遭受恶意调用。（此类常见于APP侧获取验证码）

  **建议**：在短信接口做一些加密措施（例如加入图形验证等方式），防止非法调用。

## 效果

![1561445011727](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-email/1561445011727.png)

![1561445045042](https://raw.githubusercontent.com/JankingWon/JankingWon.github.io/master/2019/aliyun-email/1561445045042.png)