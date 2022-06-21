---
title: hadoop作业查询与关闭
date: 2022-06-20 23:02:46
tags:
- hadoop
- yarn
categories: 
- 常用命令
---

# hadoop作业的查询和关闭



- ###  hadoop version <   2.3.0

查看正在运行的 Hadoop 任务：

> hadoop job -list 

关闭Hadoop 任务进程：

> hadoop job -kill $jobId 

组合以上两条命令就可以实现 kill 掉指定用户的 job

```shell
for i in `hadoop job -list | grep -w username| awk '{print $1}' | grep job_`; 
do 
	hadoop job -kill $i; 
done
```

username 就是你希望关闭 Hadoop 任务的用户



- ###  hadoop version >= 2.3.0

查看正在运行的 Hadoop 任务：

> yarn application -list 

关闭 Hadoop 任务进程：

> yarn application -kill $ApplicationId
