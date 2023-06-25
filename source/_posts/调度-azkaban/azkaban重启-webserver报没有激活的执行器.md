---
title: azkaban重启 webserver报没有激活的执行器
author: xxzuo
categories:
  - 调度-azkaban
date: 2022-07-22 21:54:31
tags:
---

azkaban 重启后 webserver报错

azkaban重启时

先重启执行器
在 /usr/local/azkaban/azkaban-exec-server 目录下执行 bin/start-exec.sh

在 /usr/local/azkaban/azkaban-web-server 目录下执行 bin/start-exec.sh

报 :ERROR [ExecutorManager] [main] [Azkaban] No activee executors found

需要去 Azkaban配置数据库中更新 port=12321的执行器激活数量为1

update executors set active = 1 where port = 12321

并且使用接口启用

curl -G "http://localhost:12321/executor?action=activate" && echo
