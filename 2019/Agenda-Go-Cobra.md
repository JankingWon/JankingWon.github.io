---
title: Agenda-Go-Cobra
urlname: Agenda-Go-Cobra
tags:
  - Go
copyright: true
category: Go
abbrlink: 40635
date: 2018-10-23 19:53:48
---

<!-- more --> 

## 1.测试

```
go get -v github.com/spf13/cobra/cobra
git clone https://golang.org/x/sys
git clone https://golang.org/x/text

//然后出现了错误
[janking@localserver cobra]$ go install ./cobra
../viper/viper.go:38:2: cannot find package "github.com/fsnotify/fsnotify" in any of:
	/usr/lib/golang/src/github.com/fsnotify/fsnotify (from $GOROOT)
	/home/janking/gowork/src/github.com/fsnotify/fsnotify (from $GOPATH)

//我用这个命令没成功，最后还是手动下载压缩包的放在上面错误提醒的相对目录就好了
git clone https://github.com/fsnotify/fsnotify

//进入相应的目录gowork/src/github.com/spf13/cobra/cobra
sudo go install ./cobra

//然后好大的一个坑，出现下列错误
[janking@localserver bin]$ ./cobra init
Error: Rel: can't make /home/janking/gowork/bin relative to 

//创建自己的github项目，然后进入里面创建agenda，在这个文件夹执行下列命令（目录gowork/src/github.com/JankingWon/agenda）
[janking@localserver agenda]$ ../../../../bin/cobra init
Your Cobra application is ready at
/home/janking/gowork/src/github.com/JankingWon/agenda

Give it a try by going there and running `go run main.go`.
Add commands to it by running `cobra add [cmdname]`.

//注册
../../../../bin/cobra add register
```

> **老师指导**：
>
> 修改 `register.go`, `init()` 添加：
>
> ```go
> registerCmd.Flags().StringP("user", "u", "Anonymous", "Help message for username")
> ```
>
> `Run` 匿名回调函数中修改为：
>
> ```go
> username, _ := cmd.Flags().GetString("user")
> fmt.Println("register called by " + username)
> ```
>
>

测试成功

![1540705248989](http://blog.janking.cn/post/Agenda-Go-Cobra/1540705248989.png)



## 2.修改

```
[janking@localserver agenda]$ ../../../../bin/cobra init
Your Cobra application is ready at
/home/janking/gowork/src/github.com/JankingWon/agenda

Give it a try by going there and running `go run main.go`.
Add commands to it by running `cobra add [cmdname]`.

//然后通过下面的办法添加命令，并修改新生成的[命令].go文件
[janking@localserver agenda]$ ../../../../bin/cobra add [命令]
//接下来就是写代码了
```



- 把`cobra.out`文件放到`/gowork/bin`里面
- 删除所有`agenda`下面所有文件
- 进入文件夹`gopath/src/github.com/[Github用户名]/agenda`文件夹

- 初始化

  `[janking@localserver agenda]$ ../../../../bin/cobra init`

- 添加命令

  `[janking@localserver agenda]$ ../../../../bin/cobra add [命令]`

- 然后修改`cmd`下面的`[命令].go`文件（如`register.go`)

- 运行测试自己的命令

  `[janking@localserver agenda]$ go run main.go register -u janking`

- 生成可执行文件

  `[janking@localserver agenda]$ go build ./`

- 就可以这样执行命令了

  `[janking@localserver agenda]$ ./agenda register -u janking`




