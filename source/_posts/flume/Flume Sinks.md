---
title: Flume-Sinks文档(进行中)
author: xxzuo
tags:
  - flume文档
categories:
  - flume
date: 2023-07-13 22:21:31
---

### Flume Sinks

#### HDFS Sink

该 Sink 将事件写入 Hadoop 分布式文件系统 (HDFS)。 它目前支持创建文本和序列文件。 它支持两种文件类型的压缩。 可以根据经过的时间或数据大小或事件数量定期滚动文件（关闭当前文件并创建新文件）。 它还按时间戳或事件起源的机器等属性对数据进行存储/分区。 HDFS 目录路径可能包含格式化转义序列，这些序列将被 HDFS 接收器替换，以生成用于存储事件的目录/文件名。 使用此Sink 需要安装 hadoop，以便 Flume 可以使用 Hadoop jar 与 HDFS 集群进行通信。 请注意，需要支持sync() 调用的Hadoop版本。

以下是支持的转义序列：


