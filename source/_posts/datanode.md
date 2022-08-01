---
title: datanode
author: xxzuo
date: 2022-07-31 14:59:16
tags:
- hdfs
categories:
- 大数据
---



# DATANODE工作机制

### DataNode启动流程

1. DataNode 启动后主动向 NameNode 注册
2. 注册成功后，NameNode会把DataNode注册在元数据中
3. 注册成功以后每周期(默认6小时)，DataNode向NameNode上报信息(块完好)
4. 心跳每3秒一次，心跳返回结果带有NameNode给DataNode的命令,比如复制数据块到另一台机器，或者删除某个数据块等等(DataNode没挂)
5. 超过10分钟+30秒 收到DataNode 的心跳，则认为该节点不可用，此时NameNode 就不会对该文件块进行读写



### DataNode数据存储

一个数据块在DataNode上以文件形式存储在磁盘上，包括两个文件

- 数据本身
- 元数据：数据块的长度、数据块的校验、时间戳



### 参数配置

DN向NN汇报当前解读信息的时间间隔，默认6小时

```xml
dfs.blockreport.intervalMsec
```



DN扫描自己节点块信息列表的时间，默认6小时

```xml
dfs.datanode.directoryscan.interval
```













