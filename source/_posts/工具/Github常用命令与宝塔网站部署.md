---
title: Github常用命令与网站部署
date: 2020-03-02T15:51:51.5
author: codezhw
summary: 介绍Git常用的命令，创建仓库分支将hexo博客自动生成代码和博客源代码分开管理，以及宝塔服务器部署Java Web项目，部署远程数据库。
categories: 工具
tags: 
   - Github
   - 服务器
   


---

# Github常用命令与网站部署



## Github常用命令



**git init 新建一个空的仓库**
**git status 查看状态**
**git add . 添加文件**
**git commit -m '注释' 提交添加的文件并备注说明**
**git remote add origin #连接你的Github仓库 连接远程仓库**
**git push -u origin master 将本地仓库文件推送到远程仓库**
**git log 查看变更日志**
**git reset --hard 版本号前六位 回归到指定版本**
**git branch 查看分支**
**git branch newname 创建一个叫newname的分支**
**git checkout newname 切换到叫newname的分支上**
**git merge newname 把newname分支合并到当前分支上**
**git pull origin master 将master分支上的内容拉到本地上**



## 使用Git向Github 提交代码



> **第一次使用**

**第一步：**在Github创建一个仓库

![创建仓库](https://i.loli.net/2020/03/02/wjiUV7frSpBd5cH.png)

**第二步：**进入本地项目文件夹。

~~~bash
git init
~~~

初始化代码仓库，会在文件夹下生成一个.git文件。

**第三步：**添加文件到版本库（只是先放在了缓存区里缓冲区）

~~~ bash
git add.
~~~

**第四步：**把添加的文件提交到版本库

~~~ bash
git commit -m "first commit" # 可以自定义提交信息
~~~

**第五步：**将本地库与远程库关联

~~~ bash
git remote add origin #后面跟上你的仓库地址
~~~

**第六步：**推送代码

~~~ bash
git push -u origin master // 推送代码
~~~

> **以后使用**



**进入要提交代码的文件**执行一下命令：

~~~ bash
git pull origin master  #拉取远程仓库到本地
git add .
git commit -m '描述内容'
git push origin master  # 推送到GitHub仓库
~~~



## Hexo博客创建分支管理



我们的Hexo博客部署在Github通过hexo clean | hexo g | hexo d 这一系列流程将我们的静态博客提交到Github仓库中。

![我的博客仓库](https://i.loli.net/2020/03/02/it1WmZXD5lofQs8.png)

这是我的Hexo博客，可以看到我们提交的是经过hexo生成的代码文件。

这和我们的博客源码文件不一样，如果哪天你更换电脑，或者误删了博客文件，会很不好管理，所以我们通过创建分支（branch）来管理我们的博客源码。



```bash
cd ###进入你的hexo博客文件
git init  #初始化过了后就不必初始化
git add . #将必要的文件依次添加
git commit -m "提交hexo 配置文件"
git branch hexo  #新建hexo分支
git checkout hexo  #切换到hexo分支上 git switch hexo也可以
git remote add origin #本地与远程对接
git push origin hexo  #push到github项目文件
```

![hexo分支](https://i.loli.net/2020/03/02/E8v9nY4xIdMi7HV.png)

这样就保存了我们的源代码的文件了。



## 宝塔将Java Web项目部署到Tomcat服务器



**准备前提：**

* 一台服务器
* 一个Java Web项目

- 宝塔服务器运维面板（傻瓜式安装）



宝塔服务器：

https://www.bt.cn/?btwaf=97644413

**进入官网按步骤操作后进入宝塔面板**

![](https://i.loli.net/2020/03/02/zmWCPj7l1vTnRAs.png)

安装这两个主要的软件（没有数据库连接可以不用，也可以用nignx）,去软件商店中安装想要的软件。

**将Java Web项目打成war包**

**点击-> `文件`->`www`->`server`>`tomcat`->`webapps`**

**将war包放到webapps目录中**

**重启tomcat服务器**

![重启tomcat](https://i.loli.net/2020/03/02/7kWTsO2YvyAB8aq.png)

**数据库配置**

**创建宝塔服务器数据库，密码会自动生成。**

![创建数据库](https://i.loli.net/2020/03/02/6EiRU2m7BpTcCjP.png)

**连接数据库的xx.properties文件改为宝塔服务器的数据库。**

**最后就可以通过：`服务器域名：端口号/项目文件名/ `  访问了。**

