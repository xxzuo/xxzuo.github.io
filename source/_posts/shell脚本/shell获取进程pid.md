---
title: shell获取进程pid
author: xxzuo
tags:
  - shell
categories:
  - shell脚本
date: 2022-08-03 23:53:27
---

### 查看进程pid

```shell
ps -ef|grep 进程名 |grep -v grep | awk '{print $2}'
```



### 判断进程是否存在

```shell
p_cnt=ps -ef|grep queue |grep -v grep| wc -l
if [ $p_cnt -le 0 ]; then
# 进程不存在
else
# 进程存在
fi
```

