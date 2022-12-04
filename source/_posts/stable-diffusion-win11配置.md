---
title: stable-diffusion(win11配置)
author: xxzuo
date: 2022-11-17 22:42:23
tags:
- ai画图
- python
categories:
---



## CUDA环境配置

![](https://raw.githubusercontent.com/xxzuo/pic_host/main/nvidia-smi.png)

```powershell
# 在 ananconda prompt 中执行 nvidia-smi 查看 cuda 版本
```

去 cuda 官网下载对应的版本

[CUDA Toolkit Archive | NVIDIA Developer](https://developer.nvidia.com/cuda-toolkit-archive)

找到对应的cuda 版本下载

![](https://raw.githubusercontent.com/xxzuo/pic_host/main/cuda1.png)

![](https://raw.githubusercontent.com/xxzuo/pic_host/main/cuda2.png)



下载之后安装 cuda



## stable diffusion环境配置

```powershell
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git
```



##### 编辑 脚本 

打开 webui-user.bat 

```powershell
@echo off

set PYTHON=
set GIT=
set VENV_DIR=
set COMMANDLINE_ARGS=

call webui.bat
```

设置 PYTHON 为 本地的 python 路径

然后 执行  webui-user.bat  

##### 问题

执行 webui-user.bat 发现

> 一直卡在Installing torch and torchvision     非常缓慢  发现是报错了



安装SSL

>  报 pip is configured with locations that require TLS/SSL, however the ssl module in Python is not available.
> 3  错误 需要安装 SSL

 https://slproweb.com/products/Win32OpenSSL.html  下载 openssl 即可



