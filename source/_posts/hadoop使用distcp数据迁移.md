---
title: hadoop使用distcp数据迁移
author: xxzuo
date: 2022-12-02 14:54:24
tags:
- hdfs
- 数据迁移
categories:
- 大数据
---

## 数据迁移

> 命令 hadoop distcp:
>
> 
> hadoop distcp -log ~/distcp_stage.log   hdfs://192.168.1.1:9000/user/hive/warehouse/ods hdfs://192.168.1.2:8020/user/hive/warehouse/
>



## 注意事项

这里的 source hdfs 和 target hdfs 必须得是 **namenode** 所在节点



## 遇到的问题

### 1.数据迁移后 hive表 count(*) = 0

可能是同步过来的数据在hive的 **metastore** 里没有行数信息,执行一次 **insert overwirte** 后就有数据了

也有可能是应用程序权限问题



### 2.分区表无数据

分区表还需要手动添加分区，`alter table xxx add if not exists partition(key1="value1", key2="value2") partition(key1="value3", key2="value4")`

hive shell 通过 `msck repair table xxx` 可以自动去读取hdfs下文件信息，来添加元数据中不存在的分区信息，但是存在jdbc连接时不识别 msck 指令的情况



### 3.客户端工具无法执行insert overwrite

datagrip执行insertoverwrite 报 return code 2 from [org.apache.hadoop.hive.ql.exec.mr](http://org.apache.hadoop.hive.ql.exec.mr/).MapRedTask

查询yarn日志发现: 找不到或无法加载主类 org.apache.hadoop.mapreduce.v2.app.MRAppMaster

但是在hive命令行中可以执行

有可能是hadoop目录权限问题
