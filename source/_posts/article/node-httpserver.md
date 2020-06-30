---
title: http-server调起本地服务
date: 2020-06-30 23:07:03
summary: 
categories: 原创
tags: 
- Node
- http-server
img:
keywords: 
- Node
- http-server
---
### 开始安装http-server：

打开cmd，输入：npm install http-server -g 

![](https://images2018.cnblogs.com/blog/830482/201807/830482-20180715220455990-1512178700.png)

### 前端如何用它启动本地服务：

进入要启动的项目所在文件夹，按住shift+右键，选择 “在此处打开命令窗口”，

出现cmd窗口，输入命令：hs -o   （等同于  http-server -open）  本地服务器就启动起来了，默认端口为8080。

ps: 可以启动多个项目，http-server会为它们分不同的端口。

### 启动的项目，点击app/就能看到界面了：

![](https://images2018.cnblogs.com/blog/830482/201807/830482-20180715220608332-1554241804.png)