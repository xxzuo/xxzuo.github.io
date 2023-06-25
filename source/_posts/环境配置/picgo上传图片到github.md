---
title: picgo上传图片到github
author: xxzuo
tags:
  - picgo
  - 图床
categories:
  - 环境配置
date: 2022-08-09 22:58:35
---







写博客的时候有很多图片需要插入，这些博客在本地的时候，可以预览到图片，但是因为图片在本地没有上传，所以一发布就看不到图片了。因为博客是部署在 **github** 上的，所以同样也用 **github** 来做图床。

**PicGo** 是一个用于快速上传图片并获取图片 URL 链接的工具，支持多个图床进行使用

### 下载picgo

下载picgo 2.3.0版本

[Release 2.3.0 · Molunerfinn/PicGo (github.com)](https://github.com/Molunerfinn/PicGo/releases/tag/v2.3.0)

windows 选择如下版本：

![](https://raw.githubusercontent.com/xxzuo/pic_host/main/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220809225734.png)



### 获取token

在 **github** 主页点击自己头像后，依次选择【Settings】->【Developer settings】->【Personal access tokens】->【Generate new token】

![](https://raw.githubusercontent.com/xxzuo/pic_host/main/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20220809230614.png)



![](https://raw.githubusercontent.com/xxzuo/pic_host/main/5136655b521491ac2bce1a586f08ac5.png)



![](https://raw.githubusercontent.com/xxzuo/pic_host/main/0cb3b7117972d880a65957e2d976622.png)



### 设置图床

选择 【GitHub图床】

![](https://raw.githubusercontent.com/xxzuo/pic_host/main/61f59a429ad79346f5224f045ba22c7.png)

仓库是 你自己 github 上的 仓库

token就是刚才获取的token
