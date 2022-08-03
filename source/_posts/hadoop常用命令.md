---
title: hadoop常用命令
author: xxzuo
date: 2022-08-01 23:10:41
tags:
- hadoop
categories:
- 大数据
---



## hdfs常用命令

查看文件列表

```bash
hadoop fs -ls <path>
```

递归查看文件

```bash
hadoop fs -ls -R <path>
```

上传文件

```bash
hadoop fs -put <localFile> <hdfsPath>
```

创建目录

```bash
hadoop fs -mkdir <path>
```

递归删除

```bash
hadoop fs -rm -r <path>
```

查看空间使用情况

```bash
hadoop fs -df -h
```

查看文件内容

```bash
hadoop fs -cat <file>
```





## yarn常用命令

查看yarn job

```bash
yarn application -list
```

kill yarn job

```bash
yarn application -kill <applicationId>
```

查看job状态

```bash
yarn application -status <applicationId>
```

