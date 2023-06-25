---
title: ubuntu20.04切换阿里源
author: xxzuo
tags:
  - ubuntu
categories:
  - 环境配置
date: 2022-07-14 00:57:22
---



### ubuntu20.04 切换阿里源

执行

> sudo apt install libopencv-dev libeigen3-dev

报错

> Err:1 http://archive.ubuntu.com/ubuntu focal/universe amd64 libfyba0 amd64 4.1.1-6build1
>   Connection failed [IP: 185.125.190.39 80]
> Err:2 http://archive.ubuntu.com/ubuntu focal/universe amd64 libfreexl1 amd64 1.0.5-3
>   Connection failed [IP: 185.125.190.36 80]
> E: Failed to fetch http://archive.ubuntu.com/ubuntu/pool/universe/f/fyba/libfyba0_4.1.1-6build1_amd64.deb  Connection failed [IP: 185.125.190.39 80]
> E: Failed to fetch http://archive.ubuntu.com/ubuntu/pool/universe/f/freexl/libfreexl1_1.0.5-3_amd64.deb  Connection failed [IP: 185.125.190.36 80]
> E: Unable to fetch some archives, maybe run apt-get update or try with --fix-missing?



怀疑可能是源的问题，因此考虑换成阿里源再试试

```bash
# 首先进入目录 备份相关文件
cd /etc/apt

cp sources.list sources.list.bak

```

开始准备换源

```bash
# 查看系统信息
lsb_release -a |grep Codename
# 结果如下
# No LSB modules are available.
# Codename:       focal
```



> 注意: Codename  :  focal 



阿里源配置模板

```shell
deb http://mirrors.aliyun.com/ubuntu/ $Codename main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ $Codename main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ $Codename-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ $Codename-security main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ $Codename-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ $Codename-updates main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ $Codename-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ $Codename-proposed main restricted universe multiverse

deb http://mirrors.aliyun.com/ubuntu/ $Codename-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ $Codename-backports main restricted universe multiverse
```

我们需要将 **$Codename** 替换为上面的 focal

替换后如下

```shell
deb http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal main restricted universe multiverse
 
deb http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-security main restricted universe multiverse
 
deb http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-updates main restricted universe multiverse
 
deb http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-proposed main restricted universe multiverse
 
deb http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ focal-backports main restricted universe multiverse
```



替换完成后

>  sudo apt-get update



再执行

> sudo apt install libopencv-dev libeigen3-dev

成功安装
