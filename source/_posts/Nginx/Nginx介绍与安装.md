---
title:  Nginx介绍与安装
date: 2020-03-15T11:44
author: codezhw
top: false
summary: 这是关于nginx简介与安装的文章。由于部署github或者gitee访问速度都不太理想，所以打算把我的hexo博客部署到阿里云服务器中，所以学习一下nginx。
categories: Nginx
tags: 
	- Linux
	- 服务器

---



# Nginx介绍与安装



## Nginx的介绍



- **什么是Nginx**

**Nginx是一个http服务器。是一个使用c语言开发的高性能的http服务器及反向代理服务器。Nginx是一款高性能的http 服务器/反向代理服务器及电子邮件（IMAP/POP3）代理服务器。由俄罗斯的程序设计师Igor Sysoev所开发，官方测试nginx能够支支撑5万并发链接，并且cpu、内存等资源消耗却非常低，运行非常稳定。**



- **Nginx的应用场景**

1. HTTP服务器，Nginx是一个http服务可以独立提供http服务。可以做网页静态服务器。（这也正是我所需要的）
2. 反向代理，就是客户访问服务器的时候，不直接访问我们的服务器而是经过一个代理服务器，去访问我们的服务器。从客户端的角度，客户不知道我服务器的信息，是通过代理服务器去进行访问。`隐藏了我们的服务器，正向代理隐藏了我们的客户端。`
3. 负载均衡，一句话：nginx会给你分配服务器压力小的去访问，对于我来说用不上。



- Nginx与Tomcat和Apache的区别

  1.相同点：都是web服务器，严格来说apache和nginx叫做http server,tomcat是application server。

  2.tomcat是一个轻量级的web服务器，作为Java应用的容（jsp,servlet），可以处理动态资源和静态资源，在高并发环境下处理能力有限，并发处理不如nginx。所以可以tomcat和nginx结合使用，tomcat处理动态资源，nginx处理静态资源，并负责并发处理。

  3.nignx相较于apache更加轻量，内存占用更少，但稳定性不如apache，不能处理动态资源。

  

  

  

## 阿里云centos7安装nginx



- **1.安装依赖**

 ```bash
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel
 ```

​	

- **2.下载并解压nginx**

```bash

# 进入local目录
cd /usr/local

# 下载1.16版本的ngxin 可以自己去选择版本
wget http://nginx.org/download/nginx-1.16.0.tar.gz

# 解压
tar -xzf nginx-1.16.0.tar.gz


```

就会在`/usr/local/` 目录下看到这两个文件

![](https://i.loli.net/2020/03/15/NeFWaH9m6UMVTJx.jpg)

- **3.安装nginx**

```bash
# 进入解压的目录
cd /usr/local/nginx-1.16.0

# 执行配置文件
./configure

# 编译并安装（nginx是C语言写的）
make
make install(或者make && make install)


```

  安装成功以后再`/usr/local/`目录下会生成一个nginx文件

![](https://i.loli.net/2020/03/15/eplGjozFk8RLXTU.jpg)

进入nginx目录会看到

![](https://i.loli.net/2020/03/15/lEBuCed7nr1OcHN.jpg)

- **4.启动nginx**

~~~bash
# 进入nginx目录
cd /usr/local/nginx/sbin

# 启动
./nginx
~~~

- **5.查看是否启动成功**

~~~bash
# 查看nginx进程
ps -ef |grep nginx

# 关闭nginx(在sbin目录下 也可以将其加入环境变量)
./nginx -s stop


~~~

![](https://i.loli.net/2020/03/15/JSe6mEcBW87lrfY.jpg)



- **6.排错以及查看结果**

> **nginx进程正常启动网页打不开问题**

nginx默认端口是：`80`

通过指令：`firewall-cmd --list-all`查看防火墙80端口是否打开，阿里云服务器开发80端口的安全组。

![](https://i.loli.net/2020/03/15/lSfwo5qQ2dXyGbz.jpg)

最后通过服务器公网IP或者虚拟机IP地址就可以访问到我们的nginx网页了。

![](https://i.loli.net/2020/03/15/8TIJulhE1Wiy43t.jpg)





