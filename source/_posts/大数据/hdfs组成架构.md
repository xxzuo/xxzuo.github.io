---
title: hdfs组成架构
author: xxzuo
tags:
  - hdfs
categories:
  - 大数据
date: 2022-07-18 22:04:52
---

### HDFS组成架构

- NameNode(nn): 就是Master, 它是一个主管，管理者
  - 管理 HDFS 的名称空间(namespace)
  - 配置副本策略
  - 管理数据块(block)映射信息
  - 处理客户端读写请求
- DataNode():就是 Slave.NameNode 下达命令，DataNode执行操作
  - 存储实际数据
  - 执行数据块 读/写 操作‘
- Client
  - 文件切分,文件上传到 HDFS 的时候，客户端将文件切分成一个一个的 Block
  - 和 NameNode 交互 获取文件位置信息
  - 和 DataNode交互，读取或者写入数据
  - 提供HDFS 管理命令
- Secondary NameNode: 并非 NameNode 的热备。当NameNode 挂掉的时候，，并不能马上替换 NameNode并提供服务
  - 辅助NameNode ，分担工作量，比如定期合并 Fsimage和 Edits,并推送给NameNode
  - 紧急情况下，可以辅助恢复 NameNode





### HDFS文件块大小

HDFS 文件在物理上是分块存储(block)，块的大小可以通过配置参数(dfs.blocksize)来规定，默认为 128M





#### HDFS块不能设置的太大也不能太小

- 如果设置太小，会增加寻址时间，程序一直在找块的开始位置
- 如果设置的太大，从磁盘传输数据的时间会明显大于定位这个块开始位置所需的时间，导致程序处理数据会变慢
