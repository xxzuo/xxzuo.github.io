---
title: shell输出重定向
author: xxzuo
tags:
  - shell
  - linux
categories:
  - shell脚本
date: 2022-06-21 22:54:27
---

## Shell输出重定向



> /dev/null

是一个特殊的设备文件，这个文件接收到任何数据都会被丢弃。因此，null 这个设备通常也被称为位桶（bit bucket）或黑洞。

常见的 重定向
>/dev/null 2>&1

含义为 :
将 **标准输出** 和 **错误输出** 都 输出到 **黑洞**
其实质是 

> 1  >  /dev/null  2>&1

- 1 代表 标准输出 
- 2 代表 错误输出
- /dev/null 2>&1 这里省略了 开头的1

表示为:
1. 原本 **1(标准输出)** 输出到 **屏幕**
2. **1>/dev/null** 将 **1(标准输出)** 输出到 **黑洞**
3. **2>&1** 将 **2(错误输出)** 输出到 **1(标准输出)**,此时因为 **1(标准输出)**  输出到 **黑洞** 了，所以 **2(错误输出)** 也输出到了 **黑洞**


以此类推
> 2>/dev/null

把 **2(错误输出)** 输出到 **黑洞**，**1(标准输出)** 打印到 **屏幕**


> 2>&1 >/dev/null

把 **2(错误输出)** 输出到 **屏幕**，**1(标准输出)** 打印到 **黑洞** 
