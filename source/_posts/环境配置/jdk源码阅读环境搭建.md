---
title: jdk源码阅读环境搭建
tags:
  - java
categories:
  - 环境配置
date: 2021-05-04 23:34:57
---

### 1.新建项目

首先新建一个普通的java项目

<img src="/img/jdk阅读环境/1.jpg">

### 2.导入jdk源码

- 在java的安装目录下找到`src.zip`这个压缩包
- 然后我们将这个压缩包复制到刚才新建的项目目录下

<img src="/img/jdk阅读环境/2.jpg">

- 右键解压缩到src目录下

<img src="/img/jdk阅读环境/3.jpg">

### 3.替换jdk关联源码

首先我们打开项目设置`open moudle settings`

点击SDKs将Sourcepath中原来关联的src给删掉

<img src="/img/jdk阅读环境/4.jpg">

然后将项目中的jdk源码关联进来

<img src="/img/jdk阅读环境/5.jpg">

设置项目jdk为刚才关联的jdk

<img src="/img/jdk阅读环境/6.jpg">

### 4.其他配置

- 在`settings`中搜索compiler 

​       修改`build project automatically`为1500 (用于解决系统资源不足)

​      <img src="/img/jdk阅读环境/11.jpg">

- 在`settings`中搜索stepping

  取消勾选

​      <img src="/img/jdk阅读环境/7.jpg">

- 删除`java.swing.plaf`下的`gtk`包

​       <img src="/img/jdk阅读环境/8.jpg">

- 引入jdk运行jar包

  打开项目设置`open moudle settings`，在`Libraries`里新增lib

  <img src="/img/jdk阅读环境/9.jpg">

  选择本机jdk安装路径下的lib文件夹

  <img src="/img/jdk阅读环境/10.jpg">