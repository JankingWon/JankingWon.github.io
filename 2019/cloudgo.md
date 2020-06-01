---
title: cloudgo
urlname: cloudgo
tags:
  - Go
copyright: true
category: Go
abbrlink: 44223
date: 2018-11-17 19:32:27
---

## net/http 部分源码分析

net/http在服务端是主要工作流程？

- 监听服务器的某个端口，端口相当于客户端与服务器端沟通的门户

- 接收到客户端的请求的时候，能够对客户端的请求进行分析处理

- 能够将需要返回的数据通过端口发送回客户端

  <!-- more --> 

1. 监听端口

    ```go
    // ListenAndServe listens on the TCP network address addr
    // and then calls Serve with handler to handle requests
    // on incoming connections.
    // Accepted connections are configured to enable TCP keep-alives.
    // Handler is typically nil, in which case the DefaultServeMux is
    // used.
    //
    // A trivial example server is:
    //
    //	package main
    //
    //	import (
    //		"io"
    //		"net/http"
    //		"log"
    //	)
    //
    //	// hello world, the web server
    //	func HelloServer(w http.ResponseWriter, req *http.Request) {
    //		io.WriteString(w, "hello, world!\n")
    //	}
    //
    //	func main() {
    //		http.HandleFunc("/hello", HelloServer)
    //		log.Fatal(http.ListenAndServe(":12345", nil))
    //	}
    //
    // ListenAndServe always returns a non-nil error.
    func ListenAndServe(addr string, handler Handler) error {
    server := &Server{Addr: addr, Handler: handler}
    return server.ListenAndServe()
    }
    ```

`addr string` 是监听的端口
`handler Handler` 是相应的处理函数
而且这两个参数还是后来被绑定到一个`Server`实例对象上，最后运行它的`ListenAndServe`方法

`server.ListenAndServe()`：

```go
// ListenAndServe listens on the TCP network address srv.Addr and then
// calls Serve to handle requests on incoming connections.
// Accepted connections are configured to enable TCP keep-alives.
// If srv.Addr is blank, ":http" is used.
// ListenAndServe always returns a non-nil error.
func (srv *Server) ListenAndServe() error {
	addr := srv.Addr
	if addr == "" {
		addr = ":http"
	}
	ln, err := net.Listen("tcp", addr)
	if err != nil {
		return err
	}
	return srv.Serve(tcpKeepAliveListener{ln.(*net.TCPListener)})
}
```

`server.ListenAndServe()`函数`srv.Serve(tcpKeepAliveListener{ln.(*net.TCPListener)})` 调用底层协议搭建服务，该函数返回一个Listener接口，然后就可以使用这个接口监听来自客户端的请求。

2. 请求分析
    使用`server(net.Listener)` 方法：

```go
// Serve accepts incoming connections on the Listener l, creating a
// new service goroutine for each. The service goroutines read requests and
// then call srv.Handler to reply to them.
//
// For HTTP/2 support, srv.TLSConfig should be initialized to the
// provided listener's TLS Config before calling Serve. If
// srv.TLSConfig is non-nil and doesn't include the string "h2" in
// Config.NextProtos, HTTP/2 support is not enabled.
//
// Serve always returns a non-nil error. After Shutdown or Close, the
// returned error is ErrServerClosed.
func (srv *Server) Serve(l net.Listener) error {
    defer l.Close()
    if fn := testHookServerServe; fn != nil {
        fn(srv, l)
    }
    var tempDelay time.Duration // how long to sleep on accept failure
    if err := srv.setupHTTP2_Serve(); err != nil {
        return err
    }
    
    srv.trackListener(l, true)
    defer srv.trackListener(l, false)
    
    baseCtx := context.Background() // base is always background, per Issue 16220
    ctx := context.WithValue(baseCtx, ServerContextKey, srv)
    for {
        rw, e := l.Accept()
        if e != nil {
            select {
            case <-srv.getDoneChan():
                return ErrServerClosed
            default:
            }
            if ne, ok := e.(net.Error); ok && ne.Temporary() {
                if tempDelay == 0 {
                    tempDelay = 5 * time.Millisecond
                } else {
                    tempDelay *= 2
                }
                if max := 1 * time.Second; tempDelay > max {
                    tempDelay = max
                }
                srv.logf("http: Accept error: %v; retrying in %v", e, tempDelay)
                time.Sleep(tempDelay)
                continue
            }
            return e
        }
        tempDelay = 0
        c := srv.newConn(rw)
        c.setState(c.rwc, StateNew) // before Serve can return
        go c.serve(ctx)
    }
} 
```
这个方法中使用`Accept()` 函数接收来自客户端的请求：`rw, e := l.Accept()；`
并将信息保存在`rw` 中，然后建立一个新的连接`c := srv.newConn(rw)；`
再创建一个`goroutine`，调用`c.server(ctx)` 来处理来自客户端的请求

## 简单使用Beego框架做小程序

- 进入go/bin
  ```
  janking@Me:~/go/bin$ ./bee new myweb
  ```

- 进入go/myweb

  ```
  janking@Me:~/go/src/myweb$ ../../bin/bee run
  ```
## 说明


| 访问符           | 功能                                               |
| ---------------- | -------------------------------------------------- |
| /hello-world     | 显示欢迎页面                                       |
| /hello-world/:id | 使用自己的id登录（显示输入的ID）                   |
| /:name           | 使用自己的用户名“登录”（显示输入的用户名）         |
| /:name/:id       | 使用自己的用户名和id“登录”（显示输入的用户名和ID） |

默认地址是`localhost:8080`


## 测试

### 浏览器测试

![1542370188173](http://blog.janking.cn/post/cloudgo/1542370188173.png)

------

![1542370274467](http://blog.janking.cn/post/cloudgo/1542370274467.png)

------

![1542372223840](http://blog.janking.cn/post/cloudgo/1542372223840.png)

------



![1542372152453](http://blog.janking.cn/post/cloudgo/1542372152453.png)

### CURL测试

这里以`/hello-world`为例

![1542370833916](http://blog.janking.cn/post/cloudgo/1542370833916.png)

### ab压力测试

![1542370975132](http://blog.janking.cn/post/cloudgo/1542370975132.png)

从请求相应时间来看

>   50%     43
>   66%     44
>   75%     47
>   80%     50
>   90%     58
>   95%     88
>   98%    102
>   99%    110
>   100%    152 (longest request)

表示50%的请求都在43ms内完成

99%的请求在110ms内完成

但是100%的请求要152ms才能完成

效率有点低

#### ab参数分析

命令行参数：

```
-n 执行的请求数量
-c 并发请求个数
-t 测试所进行的最大秒数
-p 包含了需要POST的数据的文件
-T POST数据所使用的Content-type头信息
-k 启用HTTP KeepAlive功能，即在一个HTTP会话中执行多个请求，默认时，不启用KeepAlive功能
```

显示结果参数：

- Server Software:  beegoServer:1.11.0 表示服务器使用的软件及其版本
- Server Hostname:  localhost，表示服务器主机名，请求message头部的Host字段
- Server Port:  8080，表示服务器端口号
- Document Path: 文档路径，请求message头部中的URL字段
- Document Length: 文档长度，响应头的Content-Length字段
- Concurrency Level： 并发个数，即命令中的-c参数值
- Time taken for tests: 测试花费的时间
- Complete requests: 一共完成的请求数，即命令中的-n参数值
- Failed requests: 失败的请求数
- Total transferred: 传输的字节数
- HTML transferred: 传输的HTML报文体的字节数，为Document Length * Complete requests
- Requests per second: 平均每秒的请求数，即吞吐率
- Time per request: 平均每个请求花费的时间，这是用户平均请求等待时间
- Time per request: 考虑并发时平均每个请求花费的时间，即上一个参数值除以并发数，这是服务器平均请求等待时间
- Transfer rate: 传输速率，平均每秒传输的千字节数
- Connection Times: 传输时间统计

| 参数名      | 含义     |
| ----------- | -------- |
| Connect:    | 连接时间 |
| Processing: | 处理时间 |
| Waiting:    | 等待时间 |
| Total:      | 总时间   |



[源码地址](https://github.com/JankingWon/Cloudgo)

<!-- more --> 