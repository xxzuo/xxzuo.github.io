---
title: Flume-Sinks文档(进行中)
author: xxzuo
tags:
  - flume文档
categories:
  - flume
date: 2024-03-22 22:21:31
---

### Flume Sinks

#### HDFS Sink

该 Sink 将事件写入 Hadoop 分布式文件系统 (HDFS)。 它目前支持创建文本和序列文件。 它支持两种文件类型的压缩。 可以根据经过的时间或数据大小或事件数量定期滚动文件（关闭当前文件并创建新文件）。 它还按时间戳或事件起源的机器等属性对数据进行存储/分区。 HDFS 目录路径可能包含格式化转义序列，这些序列将被 HDFS 接收器替换，以生成用于存储事件的目录/文件名。 使用此Sink 需要安装 hadoop，以便 Flume 可以使用 Hadoop jar 与 HDFS 集群进行通信。 请注意，需要支持sync() 调用的Hadoop版本。

以下是支持的转义序列：

| 别名 | 描述 |
|---|---|
| `%{host}` | 替换事件头部名为“host”的值。支持任意头部名称。 |
| `%t` | Unix时间，以毫秒为单位 |
| `%a` | 地区设置的星期几简称 (Mon, Tue, ...) |
| `%A` | 地区设置的星期几全称 (Monday, Tuesday, ...) |
| `%b` | 地区设置的月份简称 (Jan, Feb, ...) |
| `%B` | 地区设置的月份全称 (January, February, ...) |
| `%c` | 地区设置的日期和时间 (Thu Mar 3 23:05:25 2005) |
| `%d` | 月份中的日 (01) |
| `%e` | 月份中的日，不补零 (1) |
| `%D` | 日期；同 `%m/%d/%y` |
| `%H` | 小时 (00..23) |
| `%I` | 小时 (01..12) |
| `%j` | 一年中的日 (001..366) |
| `%k` | 小时 ( 0..23) |
| `%m` | 月份 (01..12) |
| `%n` | 月份，不补零 (1..12) |
| `%M` | 分钟 (00..59) |
| `%p` | 地区设置的上午或下午的等效表示 |
| `%s` | 自1970-01-01 00:00:00 UTC以来的秒数 |
| `%S` | 秒 (00..59) |
| `%y` | 年份的后两位 (00..99) |
| `%Y` | 年份 (2010) |
| `%z` | +hhmm 数字时区表示法 (例如, -0400) |
| `%[localhost]` | 替换代理运行所在主机的主机名 |
| `%[IP]` | 替换代理运行所在主机的IP地址 |
| `%[FQDN]` | 替换代理运行所在主机的规范主机名 |

注意：转义字符串 `%[localhost]`、`%[IP]` 和 `%[FDN]` 都依赖于Java获取主机名的能力，这在某些网络环境中可能会失败。
正在使用的文件的名称将被修改为在末尾包含“.tmp”。关闭文件后，此扩展名将被删除。这允许排除目录中部分完整的文件。必填属性以**粗体**显示。

> 注意：对于所有与时间相关的转义序列，事件的标头中必须存在一个关键字为“timestamp”的标头（除非hdfs.useLocalTimeStamp设置为true）。一种自动添加的方法是使用TimestampInterceptor。


|名称|默认值|描述|
|---|---|---|
|**channel**| – | - |
|**type**| – | 组件类型名称，需要是hdfs |
|**hdfs.path**| – | HDFS目录路径（例如 hdfs://namenode/flume/webdata/）|
|hdfs.filePrefix| FlumeData | Flume在HDFS目录中创建的文件名前缀 |
|hdfs.fileSuffix| – | 附加到文件的后缀（例如 .avro - 注意：句号不会自动添加）|
|hdfs.inUsePrefix| – | 用于Flume正在积极写入的临时文件的前缀 |
|hdfs.inUseSuffix| .tmp | 用于Flume正在积极写入的临时文件的后缀 |
|hdfs.emptyInUseSuffix| false | 如果为false，则在写入输出时使用hdfs.inUseSuffix。关闭输出后，hdfs.inUseSuffix从输出文件名中移除。如果为true，则忽略hdfs.inUseSuffix参数，改为使用空字符串。|
|hdfs.rollInterval| 30 | 滚动当前文件前等待的秒数（0 = 永不根据时间间隔滚动）|
|hdfs.rollSize| 1024 | 触发滚动的文件大小，以字节为单位（0: 永不根据文件大小滚动）|
|hdfs.rollCount| 10 | 写入文件的事件数量，之后滚动（0 = 永不根据事件数量滚动）|
|hdfs.idleTimeout| 0 | 不活跃文件关闭的超时时间（0 = 禁用空闲文件的自动关闭）|
|hdfs.batchSize| 100 | 在文件刷新到HDFS之前写入的事件数量 |
|hdfs.codeC| – | 压缩编解码器。以下之一：gzip, bzip2, lzo, lzop, snappy |
|hdfs.fileType| SequenceFile | 文件格式：目前是SequenceFile, DataStream 或 CompressedStream (1)DataStream不会压缩输出文件，请不要设置codeC (2)CompressedStream需要设置hdfs.codeC，并使用可用的codeC |
|hdfs.maxOpenFiles| 5000 | 允许打开的文件数量。如果超过此数量，最旧的文件将被关闭。|
|hdfs.minBlockReplicas| – | 指定每个HDFS块的最小副本数。如果没有指定，则来自类路径中默认的Hadoop配置。|
|hdfs.writeFormat| Writable | 序列文件记录的格式。Text 或 Writable 之一。在创建数据文件之前设置为Text，否则这些文件无法被Apache Impala (孵化中) 或 Apache Hive 读取。|
|hdfs.threadsPoolSize| 10 | 每个HDFS sink用于HDFS IO操作（打开、写入等）的线程数 |
|hdfs.rollTimerPoolSize| 1 | 每个HDFS sink用于调度定时文件滚动的线程数 |
|hdfs.kerberosPrincipal| – | 用于访问安全HDFS的Kerberos用户主体 |
|hdfs.kerberosKeytab| – | 用于访问安全HDFS的Kerberos keytab |
|hdfs.proxyUser| – | - |
|hdfs.round| false | 时间戳是否应该向下舍入（如果为true，影响所有基于时间的转义序列，除了 %t）|
|hdfs.roundValue| 1 | 向下舍入到此的最近倍数（使用hdfs.roundUnit配置的单位），小于当前时间。|
|hdfs.roundUnit| second | 向下舍入值的单位 - second, minute 或 hour。|
|hdfs.timeZone| Local Time | 用于解析目录路径的时区名称，例如 America/Los_Angeles。|
|hdfs.useLocalTimeStamp| false | 在替换转义序列时使用本地时间（而不是事件头部的时间戳）。|
|hdfs.closeTries| 0 | sink在尝试关闭文件后必须重命名文件的次数。如果设置为1，这个sink在重命名失败（例如，由于NameNode或DataNode故障）后不会重试，可能会留下带有.tmp扩展名的文件处于打开状态。如果设置为0，sink将不断尝试重命名文件，直到文件最终被重命名（没有尝试次数的限制）。文件可能仍然保持打开状态，如果关闭调用失败，但数据将完整，这种情况下，文件只有在Flume重启后才会关闭。|
|hdfs.retryInterval| 180 | 关闭文件的连续尝试之间的时间（秒）。每次关闭调用都会花费多个RPC往返到Namenode，所以设置得太低可能会导致Namenode负载过大。如果设置为0或更低，sink在第一次尝试失败后不会尝试关闭文件，可能会留下打开状态的文件或带有“.tmp”扩展名。|
|serializer| TEXT | 其他可能的选项包括 avro_event 或 EventSerializer.Builder 接口实现的完全限定类名。|
|serializer.*| |||

