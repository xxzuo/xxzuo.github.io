---
title: hadoop磁盘空间清理
tags:
  - hadoop
  - linux
categories:
  - 大数据
date: 2022-06-20 23:02:59
---



查看linux磁盘空间大文件

> du -h / --max-depth=5 | sort -hr | head -n 10

查看hadoop大文件

> hdfs dfs -du -h  /

分析其中占用空间过多的文件是否可以删除
