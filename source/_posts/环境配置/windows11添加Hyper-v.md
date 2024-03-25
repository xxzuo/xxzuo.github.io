---
title: windows11添加Hyper-v
author: xxzuo
tags:
  - windows
categories:
  - 环境配置
date: 2024-03-21 09:17:15
---


win11 家庭版没有 Hyper-v, 需要手动添加

hyper-v.cmd

```cmd
pushd "%~dp0"

dir /b %SystemRoot%\servicing\Packages\*Hyper-V*.mum > hyper-v.txt

for /f %%i in ('findstr /i . hyper-v.txt 2^>nul') do dism /online /norestart /add-package:"%SystemRoot%\servicing\Packages\%%i"

del hyper-v.txt

Dism /online /enable-feature /featurename:Microsoft-Hyper-V-All /LimitAccess /ALL
```


执行这个脚本即可