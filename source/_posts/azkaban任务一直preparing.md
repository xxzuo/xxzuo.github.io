---
title: azkaban任务一直preparing
author: xxzuo
date: 2022-07-22 21:53:47
tags:
categories:
- azkaban
---



内存问题：
过滤器会检查 executor 主机空余内存是否会大于 6G，如果不足 6G，则 web-server 会认为 集群资源不够， 不会将任务交由该主机执行，需要修改 azkaban-web下的azkaban.properties配置文件，去掉MinimumFreeMemory

```properties
# 原来
azkaban.executorselector.filters=StaticRemainingFlowSize,MinimumFreeMemory,CpuStatus

# 新
azkaban.executorselector.filters=StaticRemainingFlowSize,CpuStatus
```
