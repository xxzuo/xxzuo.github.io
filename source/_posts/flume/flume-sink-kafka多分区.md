---
title: flume-sink-kafka多分区
author: xxzuo
tags:
  - flume
  - kafka
categories:
  - flume
date: 2022-11-17 22:41:51
---

# 问题

如果不做任何设置，flume 在写 kafka时只会写到一个分区，由于kafka 的一个分区 对于一个 消费者组来说只能有一个消费者

这样会影响消费速度，所以想flume 在 写 kafka 时就写到多个分区中

# 解决方法



官方文档中说明 flume 写 kafka的分区 是根据 FlumeEvent 的 headers 的key 来判断写入哪个分区的，如果 key 为 null 会随机分区

所以 我们需要 添加拦截器，给headers 中添加一个 key

设置 flume 配置文件如下

> a1.sources.r1.interceptors = i1
>
> a1.sources.r1.interceptors.i1.type = org.apache.flume.sink.solr.morphline.UUIDInterceptor$Builder
> a1.sources.r1.interceptors.i1.headerName=key
> a1.sources.r1.interceptors.i1.preserveExisting=false
