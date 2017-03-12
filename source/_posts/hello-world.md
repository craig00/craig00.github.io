---
title: Hello World
date: 2017-02-28 10:16:58
categories: hexo
tags: [hexo]
---
用hexo在github上搭建个人博客，本来挺简单，但是不动手试试就永远不知道自己到底会在什么地方犯糊涂。


### 安装nodejs和git

上官网下载最新版，配置一下环境变量即可。（程序员基本技能）
在github官网上创建一个repository，名称比较特殊，必须是yourname.github.io不然就会失败。

### 安装hexo

打开git bash，在合适的文件夹目录中执行以下命令
``` bash
$ npm install hexo-cli -g
$ hexo init blog
$ cd blog
$ npm install
$ hexo server
```
在浏览器上成功访问localhost:4000的话，那就说明hexo安装成功了。然而我就碰到了一个问题，浏览器一直在转圈圈，没有任何页面显示。这个问题是因为电脑上装了福昕阅读器，端口冲突了。设置其他端口进行访问即可。
``` bash
$ hexo s -p 4001
```


### 更换hexo主题

都说next主题好看，所以跟风尝试了下。首先去github下载对应的主题包，直接git命令行下载实在太慢了，解压缩后放到themes文件夹内，修改文件名为next。在blog\_config.yml中找到‘theme’，更改其值为next。这个文件的格式要求很严厉，好像不能随便缩进，而且键值对中间的冒号后面必须跟着空格。以下命令是清除静态资源，重新生成静态资源，启动服务。
``` bash
$ hexo clean
$ hexo g
$ hexo s
```


### 部署hexo发布到github

首先可以是同步代码
``` bash
$ git init  
$ git add .
$ git commit -m "init" 
$ git remote add origin 项目的github地址 
$ git pull origin master  
$ git push -u origin master
```
在blog\_config.yml中找到deploy，增加一点内容

``` bash
deploy:
  type: git
  repo: https://github.com/craig00/craig00.github.io.git
  branch: master
```
然后下载安装一个扩展，执行hexo d命令即可。
``` bash
$ npm install hexo-deployer-git --save
$ hexo d
```