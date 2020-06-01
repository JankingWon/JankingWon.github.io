---
title: 使用NGINX反向代理部署Spring Boot应用
urlname: nginx-springboot
tags:
  - Nginx
  - SpringBoot
copyright: false
category: Service
date: 2019-05-25 12:57:33
---

## 什么是Spring Boot

[Spring Boot](https://projects.spring.io/spring-boot/)通过大量的默认配置，让使用[Spring框架](https://projects.spring.io/spring-framework/)进行开发变得方便快捷，从而使得Java开发人员专注于程序原型设计。本文介绍如何创建一个简单的Spring Boot应用，然后通过NGINX反向代理进行发布。

## 开始之前

你需要一个同时装有Java 8和NGINX的Linode虚拟主机。如果这些已经安装好了，请跳过此步。

<!-- more --> 

### 安装Java JDK 8

1. 安装`software-properties-common`：

```bash
sudo apt install software-properties-common
```

2. 添加Oracle PPA仓库：

```bash
sudo apt-add-repository ppa:webupd8team/java
```

3. 更新所有已安装的软件包：

```bash
sudo apt update
```

4. 安装Oracle JDK。若需要安装Java 9 JDK，将`java8`改为`java9`的命令为：

```bash
sudo apt install oracle-java8-installer
```

5. 检查Java版本：

```bash
java -version
```

### 安装NGINX

以下步骤介绍在Ubuntu上安装从NGINX官方库下载的NGINX开发版。需要了解NGINX其他发行版，请参阅[NGINX管理指南](https://docs.nginx.com/nginx/admin-guide/installing-nginx/installing-nginx-open-source/#installing-a-prebuilt-package)。若需要了解如何为生产环境配置NGINX，请参阅[*NGINX入门指南*](https://www.linode.com/docs/web-servers/nginx/nginx-installation-and-basic-setup/)。

在文本编辑器中打开`/etc/apt/sources.list`，并在文件最后追加以下内容。再将`CODENAME`替换为Ubuntu版本的代号。例如，为Ubuntu 18.04，代号为Bionic Beaver（仿生海狸），以下使用`bionic`替代`CODENAME`：

```bash
/etc/apt/sources.list1

deb http://nginx.org/packages/mainline/ubuntu/ CODENAME nginx
```

2. 导入仓库的包签名密钥并将其添加到`apt`：

```bash
sudo wget http://nginx.org/keys/nginx_signing.key 
sudo apt-key add nginx_signing.key
```

3. 安装NGINX：

```bash
sudo apt update 
sudo apt install nginx
```

4. 确保NGINX正在运行并设置开机自启动：

```bash
sudo systemctl start nginx 
sudo systemctl enable nginx
```

## 安装Spring Boot CLI

Spring Boot CLI使得搭建项目框架变得更加容易。[SDKMAN!](http://sdkman.io/)是一个简化Spring CLI安装和构建的工具（如同Gradle或Maven）。使用Spring Boot CLI，可以直接在命令行中创建新项目。

1. 安装SDKMAN!的依赖项：

```bash
sudo apt install unzip zip
```

2. 安装SDKMAN!：

```bash
curl -s https://get.sdkman.io | bash
```

3. 按照控制台中打印的说明操作：

```bash
source "/home/username/.sdkman/bin/sdkman-init.sh"
```

验证SDKMAN!是否已安装：

```bash
sdk help
```

4. 安装Spring CLI：

```bash
sdk install springboot
```

验证安装：

```js
spring version
```

5. 安装Gradle：

```bash
sdk install gradle 4.5.1
Downloading: gradle 4.5.1
  
In progress...
  
######################################################################## 100.0%
  
Installing: gradle 4.5.1 
Done installing!
```

## 构建一个jar文件

有许多可用的构建工具。Spring Boot CLI默认使用Maven，但本文中将使用Gradle。想了解有关Maven和Gradle之间差异的讨论，请参阅[Maven和Gradle比较](https://gradle.org/maven-vs-gradle/)。

1. 使用Spring Boot CLI创建一个新项目。使用项目框架创建一个名为`hello-world`的新目录。

```bash
spring init --build=gradle --dependencies=web --name=hello hello-world
```

注意要查看Spring Boot CLI的可能参数的完整列表，请运行：

```bash
spring init --list
```

2. 进入项目目录。此示例创建一个Endpoint以在Spring应用中返回"Hello world"。需添加两个额外的引用和一个新类。

```java
~/hello-world/src/main/java/com/example/helloworld/HelloApplication.java
 
package com.example.helloworld;  
import org.springframework.boot.SpringApplication; 
import org.springframework.boot.autoconfigure.SpringBootApplication; 
import org.springframework.web.bind.annotation.RestController; 
import org.springframework.web.bind.annotation.RequestMapping;  

@SpringBootApplication 
public class HelloApplication {         
       public static void main(String[] args) {                 
         SpringApplication.run(HelloApplication.class, args);         
       } 
}

@RestController 
class Hello {     
   @RequestMapping("/")     
   String index() {         
      return "Hello world";     
   } 
}
```

3. 构建应用程序。这将在在项目中创建一个名为`build`的新目录。

```bash
./gradlew build
```

4. 运行嵌入了Tomcat服务器的应用。该应用将在`localhost:8080`上运行。可以按`Ctrl+C`停止。

```bash
java -jar build/libs/hello-world-0.0.1-SNAPSHOT.jar
```

5. 应用可以在不链接jar文件的情况下运行。

```bash
gradle bootRun
```

6. 测试应用是否正确地在`localhost`中运行：

```bash
curl localhost:8080
Hello world
```

7. 按`CTRL+C`停止Tomcat服务器。

## 创建一个初始化脚本

将Spring Boot应用设置为服务以在服务器重启时自启动：

```bash
/etc/systemd/system/helloworld.service

[Unit] 
Description=Spring Boot HelloWorld 
After=syslog.target 
After=network.target[Service] 
User=username 
Type=simple  

[Service] 
ExecStart=/usr/bin/java -jar /home/linode/hello-world/build/libs/hello-world-0.0.1-SNAPSHOT.jar 
Restart=always 
StandardOutput=syslog 
StandardError=syslog 
SyslogIdentifier=helloworld  

[Install] 
WantedBy=multi-user.target
```

启动服务：

```bash
sudo systemctl start helloworld
```

检查状态是否生效：

```bash
sudo systemctl status helloworld
```

## 反向代理

现在Spring应用已作为服务运行，NGINX代理允许将应用部署到非特权端口并设置SSL。

1. 为反向代理创建NGINX配置文件：

```config
/etc/nginx/conf.d/helloworld.conf 

server {        
   listen 80;        
   listen [::]:80;  
           
   server_name example.com;    
         
   location / {             
           proxy_pass http://localhost:8080/;              
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;              
           proxy_set_header X-Forwarded-Proto $scheme;              
           proxy_set_header X-Forwarded-Port $server_port;         
   } 
}
```

2. 测试配置以确保无误：

```bash
sudo nginx -t
```

3. 如果没有错误，请重新启动NGINX以使更改生效：

```bash
sudo systemctl restart nginx
```

4. 现在可以通过浏览器访问该应用。访问Linode的公共IP地址，应显示"Hello world"消息。

## 效果

![1561444559684](https://blog.janking.cn/post/nginx-springboot/1561444559684.png)

![1561444619964](https://blog.janking.cn/post/nginx-springboot/1561444619964.png)