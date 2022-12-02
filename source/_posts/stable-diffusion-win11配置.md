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



执行 webui.bat 发现

> 一直卡在Installing torch and torchvision     非常缓慢



准备手动下载 torch

> 去 torch官网 [Start Locally | PyTorch](https://pytorch.org/get-started/locally/) 手动下载
>
> pip install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu117





安装SSL





进入 venv环境

