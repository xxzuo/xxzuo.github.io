---
title: anaconda配置国内镜像
author: xxzuo
tags:
  - python
categories:
  - 环境配置
date: 2022-11-15 23:06:24
---

# Anaconda配置国内镜像



- 查看已有镜像

```bash
conda config --show channels

channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/msys2/
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
  - defaults

```

- 删除配置

```bash
# 删除所有镜像
conda config --remove-key channels

# 删除指定镜像
conda config --remove channels [urls]
```

- 配置国内镜像

```bash
conda config --add channels http://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/

conda config --set show_channel_urls yes
```

