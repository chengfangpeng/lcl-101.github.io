---
layout:     post
title:      "Ubuntu安装Fiddler"
subtitle:   " \"Ubuntu安装Fiddler\""
date:       2016-02-21 12:00:00
author:     "X-ray"
header-img: "img/post-bg-2015.jpg"
tags:
    - Ubuntu
---


前言
--

Fiddler 是Windows上一个非常强大的抓包工具，其实在Ubuntu上也是可以安装的，具体的方法如下：

安装mono
------
fiddler是基于.net开发，所以安装fidder前需要安装.net framework。而.net framework 只能运行在windows，针对linux和mac操作系统，可通过安装mono framework 来支持fiddler运行。
安装命令：

```
sudo apt-get install  mono-complete
```

下载Fiddler
---------
[下载地址](http://fiddler.wikidot.com/mono)
下载Fiddler for Mono Current Linux build
解压

运行Fiddler
---------
进入解压目录直接运行

```
./fiddler.exe
```
就可以使用了。