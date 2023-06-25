---
title: 合并flink写入hive产生的小文件
author: xxzuo
tags:
  - flink
  - hive
categories:
  - 大数据
date: 2022-08-04 00:18:29
---



flink 写入数据到 hdfs 时，会产生很多的小文件，因为每个文件均按块存储,每个块的元数据存储在NameNode的内存中,因此*HDFS*存储*小文件*会非常低效。因为大量的*小文件*会耗尽NameNode中的大部分内存。

因为我们需要将 小文件合并。

只需要利用 HIVE 的 MR 即可，也就是执行 这条SQL 

```sql
insert overwrite table hive_table
select * from hive_table where partition_col = ''
```

其中 **partition_col**是表的分区字段，这样，该分区内的小文件会自动合并

