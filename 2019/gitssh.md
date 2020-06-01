---
title: Git添加多个SSH密钥连接远程库
urlname: gitssh
tags:
  - Git
copyright: true
category: Git
date: 2019-01-18 15:55:16
---

## 生成RSA密钥对

输入下列命令生成`gitee`的密钥

<!-- more --> 

```bash
$ ssh-keygen -t rsa -C "1634363254@qq.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/d/MyBlog/.ssh/id_rsa): /c/users/janking/.ssh/gitee_id_rsa
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/users/janking/.ssh/gitee_id_rsa.
Your public key has been saved in /c/users/janking/.ssh/gitee_id_rsa.pub.
The key fingerprint is:
SHA256:u5Qymut+JLvtu1ohlSuvw9c+tlul0xtjvcsEfh8YMME 1634363254@qq.com
The key's randomart image is:
+---[RSA 2048]----+
|          ..     |
|       .   E.    |
|      o    o     |
|     . .    o    |
|    o o S   o.   |
|    .+.. o = oo  |
|   . +=.+ + *.+. |
|    +*o++o o B o.|
```

输入下列命令生成`github`的密钥

```bash
$ ssh-keygen -t rsa -C "1634363254@qq.com"
Generating public/private rsa key pair.
Enter file in which to save the key (/d/MyBlog/.ssh/id_rsa): /c/users/janking/.ssh/github_id_rsa
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /c/users/janking/.ssh/github_id_rsa.
Your public key has been saved in /c/users/janking/.ssh/github_id_rsa.pub.
The key fingerprint is:
SHA256:yJmWkhvY5pVeERHu1Y/zUHd03w58nIVrHwpp/pN1nq4 1634363254@qq.com
The key's randomart image is:
+---[RSA 2048]----+
|        +o     .+|
|       . . . ..o*|
|        o . o +oB|
|   o o B o + +o*.|
|  . * X S o =.o.o|
|   o B .   . = .o|
|    o .     . +.o|
|             + ..|
```

## 添加公钥

然后把`gitee_id_rsa.pub`和`github_id_rsa.pub`里的**全部内容**分别添加到`Gitee`和`Github`的`SSH`公钥列表中

### gitee添加公钥

![1547799308535](http://blog.janking.cn/post/gitssh/1547799308535.png)

### github添加公钥

![1547799405722](http://blog.janking.cn/post/gitssh/1547799405722.png)

## 启用私钥！

### 方法一、使用config文件配置

在`/c/users/janking/.ssh/`目录下新建一个文件`config`（没有后缀）

类似于下面的写法

```
# gitee
Host gitee.com
    HostName git
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/gitee_id_rsa
# github
Host github.com
    HostName git
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/github_id_rsa
```

这种方法是可以的，但是这个`config`文件总是写不对，上面的是已经验证过的正确的文件



>  之前一直不可以是因为我的环境变量被我弄坏了
>
> `Win10`的环境变量里面没有`HOME`，我私自加了个`HOME`，所以放的地方就不对
>
> 所以要记得把`.ssh`放到环境变量`HOME`下面，如果没有这个变量，默认就是用户名目录（如上）
>

### 方法二、临时添加验证

输入

```
$ ssh-add gitee_id_rsa
```



> 如果报下面的错误
>
> `Could not open a connection to your authentication agent.`
> 
> 输入
>
> `$ ssh-agent bash`
>
> 再输入上面的命令



会显示

```
Identity added: gitee_id_rsa (1634363254@qq.com)
```

说明添加成功！

> 不过这个方法的缺点就是不能永久保存.....所以然并卵
>

### 方法三、使用同一个密钥

**这是最简单的办法**

就生成一个密钥，默认命名为`id_rsa`

> 这个名字不能改，不然可能不能识别

然后把公钥同时放到`github`和`gitee`上就好了！

因为密钥对公钥可以**一对多**

## 测试连通性

不管用哪种方法，可以先通过`ssh`命令测试一下连通性，如下是成功的结果

```bash
$ ssh -T gitee.com
Hi 王建! You've successfully authenticated, but GITEE.COM does not provide shell access.

$ ssh -T github.com
The authenticity of host 'github.com (13.229.188.59)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'github.com,13.229.188.59' (RSA) to the list of known hosts.
Hi JankingWon! You've successfully authenticated, but GitHub does not provide shell access.
```

## 查看所有已添加的密钥

```bash
$ ssh-add -l
```


