---
title: Doris-Flink-Connector
author: xxzuo
tags:
  - flink
  - doris
categories:
  - flink-cdc
date: 2024-01-10 19:26:00
---

来源  https://doris.incubator.apache.org/zh-CN/docs/ecosystem/flink-doris-connector

[Flink Doris Connector](https://github.com/apache/doris-flink-connector) 可以支持通过 Flink 操作（读取、插入、修改、删除） Doris 中存储的数据。本文档介绍Flink如何通过Datastream和SQL操作Doris。

> **注意：**
> 
> 1. 修改和删除只支持在 Unique Key 模型上
> 2. 目前的删除是支持 Flink CDC 的方式接入数据实现自动删除，如果是其他数据接入的方式删除需要自己实现。Flink CDC 的数据删除使用方式参照本文档最后一节

## 版本兼容

|Connector Version|Flink Version|Doris Version|Java Version|Scala Version|
|---|---|---|---|---|
|1.0.3|1.11+|0.15+|8|2.11,2.12|
|1.1.1|1.14|1.0+|8|2.11,2.12|
|1.2.1|1.15|1.0+|8|-|
|1.3.0|1.16|1.0+|8|-|
|1.4.0|1.15,1.16,1.17|1.0+|8|-|
## 使用
### Maven

添加 flink-doris-connector

```
<!-- flink-doris-connector -->
<dependency>  
	<groupId>org.apache.doris</groupId>  
	<artifactId>flink-doris-connector-1.16</artifactId>  
	<version>1.4.0</version>
</dependency>  
```

**备注**
1.请根据不同的 Flink 版本替换对应的 Connector 和 Flink 依赖版本。

2.也可从[这里](https://repo.maven.apache.org/maven2/org/apache/doris/)下载相关版本jar包。


### 编译

编译时，可直接运行`sh build.sh`，具体可参考[这里](https://github.com/apache/doris-flink-connector/blob/master/README.md)。

编译成功后，会在 `dist` 目录生成目标jar包，如：`flink-doris-connector-1.5.0-SNAPSHOT.jar`。 将此文件复制到 `Flink` 的 `classpath` 中即可使用 `Flink-Doris-Connector` 。例如， `Local` 模式运行的 `Flink` ，将此文件放入 `lib/` 文件夹下。 `Yarn` 集群模式运行的 `Flink` ，则将此文件放入预部署包中。


## 配置

^2f987c

### 通用配置项

|Key|Default Value|Required|Comment|
|---|---|---|---|
|fenodes|--|Y|Doris FE http 地址， 支持多个地址，使用逗号分隔|
|benodes|--|N|Doris BE http 地址， 支持多个地址，使用逗号分隔，参考[#187](https://github.com/apache/doris-flink-connector/pull/187)|
|table.identifier|--|Y|Doris 表名，如：db.tbl|
|username|--|Y|访问 Doris 的用户名|
|password|--|Y|访问 Doris 的密码|
|doris.request.retries|3|N|向 Doris 发送请求的重试次数|
|doris.request.connect.timeout.ms|30000|N|向 Doris 发送请求的连接超时时间|
|doris.request.read.timeout.ms|30000|N|向 Doris 发送请求的读取超时时间|

### Source 配置项

|Key|Default Value|Required|Comment|
|---|---|---|---|
|doris.request.query.timeout.s|3600|N|查询 Doris 的超时时间，默认值为1小时，-1表示无超时限制|
|doris.request.tablet.size|Integer. MAX_VALUE|N|一个 Partition 对应的 Doris Tablet 个数。 此数值设置越小，则会生成越多的 Partition。从而提升 Flink 侧的并行度，但同时会对 Doris 造成更大的压力。|
|doris.batch.size|1024|N|一次从 BE 读取数据的最大行数。增大此数值可减少 Flink 与 Doris 之间建立连接的次数。 从而减轻网络延迟所带来的额外时间开销。|
|doris.exec.mem.limit|2147483648|N|单个查询的内存限制。默认为 2GB，单位为字节|
|doris.deserialize.arrow.async|FALSE|N|是否支持异步转换 Arrow 格式到 flink-doris-connector 迭代所需的 RowBatch|
|doris.deserialize.queue.size|64|N|异步转换 Arrow 格式的内部处理队列，当 doris.deserialize.arrow.async 为 true 时生效|
|doris.read.field|--|N|读取 Doris 表的列名列表，多列之间使用逗号分隔|
|doris.filter.query|--|N|过滤读取数据的表达式，此表达式透传给 Doris。Doris 使用此表达式完成源端数据过滤。比如 age=18。|

### Sink 配置项

^12f4d6

|Key|Default Value|Required|Comment|
|---|---|---|---|
|sink.label-prefix|--|Y|Stream load导入使用的label前缀。2pc场景下要求全局唯一 ，用来保证Flink的EOS语义。|
|sink.properties.*|--|N|Stream Load 的导入参数。  <br>例如: 'sink.properties.column_separator' = ', ' 定义列分隔符， 'sink.properties.escape_delimiters' = 'true' 特殊字符作为分隔符,'\x01'会被转换为二进制的0x01  <br>  <br>JSON格式导入  <br>'sink.properties.format' = 'json' 'sink.properties.read_json_by_line' = 'true'  <br>详细参数参考[这里](https://doris.incubator.apache.org/zh-CN/docs/data-operate/import/import-way/stream-load-manual)。|
|sink.enable-delete|TRUE|N|是否启用删除。此选项需要 Doris 表开启批量删除功能(Doris0.15+版本默认开启)，只支持 Unique 模型。|
|sink.enable-2pc|TRUE|N|是否开启两阶段提交(2pc)，默认为true，保证Exactly-Once语义。关于两阶段提交可参考[这里](https://doris.incubator.apache.org/zh-CN/docs/data-operate/import/import-way/stream-load-manual)。|
|sink.buffer-size|1MB|N|写数据缓存buffer大小，单位字节。不建议修改，默认配置即可|
|sink.buffer-count|3|N|写数据缓存buffer个数。不建议修改，默认配置即可|
|sink.max-retries|3|N|Commit失败后的最大重试次数，默认3次|

### Lookup Join 配置项

|Key|Default Value|Required|Comment|
|---|---|---|---|
|jdbc-url|--|Y|jdbc连接信息|
|lookup.cache.max-rows|-1|N|lookup缓存的最大行数，默认值-1，不开启缓存|
|lookup.cache.ttl|10s|N|lookup缓存的最大时间，默认10s|
|lookup.max-retries|1|N|lookup查询失败后的重试次数|
|lookup.jdbc.async|false|N|是否开启异步的lookup，默认false|
|lookup.jdbc.read.batch.size|128|N|异步lookup下，每次查询的最大批次大小|
|lookup.jdbc.read.batch.queue-size|256|N|异步lookup时，中间缓冲队列的大小|
|lookup.jdbc.read.thread-size|3|N|每个task中lookup的jdbc线程数|

## Doris 和 Flink 列类型映射关系

|Doris Type|Flink Type|
|---|---|
|NULL_TYPE|NULL|
|BOOLEAN|BOOLEAN|
|TINYINT|TINYINT|
|SMALLINT|SMALLINT|
|INT|INT|
|BIGINT|BIGINT|
|FLOAT|FLOAT|
|DOUBLE|DOUBLE|
|DATE|DATE|
|DATETIME|TIMESTAMP|
|DECIMAL|DECIMAL|
|CHAR|STRING|
|LARGEINT|STRING|
|VARCHAR|STRING|
|DECIMALV2|DECIMAL|
|TIME|DOUBLE|
|HLL|Unsupported datatype|

## Flink 写入指标

其中Counter类型的指标值为导入任务从开始到当前的累加值，可以在Flink Webui metrics中观察各表的各项指标。

| Name | Metric Type | Description |
| ------------------------- | ----------- | ------------------------------------------ |
| totalFlushLoadBytes | Counter | 已经刷新导入的总字节数 |
| flushTotalNumberRows | Counter | 已经导入处理的总行数 | 
| totalFlushLoadedRows | Counter | 已经成功导入的总行数 | 
| totalFlushTimeMs | Counter | 已经成功导入完成的总时间 | 
| totalFlushSucceededNumber | Counter | 已经成功导入的次数 | 
| totalFlushFailedNumber | Counter | 失败导入 的次数 | 
| totalFlushFilteredRows | Counter | 数据质量不合格的总行数 | 
| totalFlushUnselectedRows | Counter | 被 where 条件过滤的总行数 | 
| beginTxnTimeMs | Histogram | 向Fe请求开始一个事务所花费的时间，单位毫秒 | 
| putDataTimeMs | Histogram | 向Fe请求获取导入数据执行计划所花费的时间 |
| readDataTimeMs | Histogram | 读取数据所花费的时间 | 
| writeDataTimeMs | Histogram | 执行写入数据操作所花费的时间 |
| commitAndPublishTimeMs | Histogram | 向Fe请求提交并且发布事务所花费的时间 | 
| loadTimeMs | Histogram | 导入完成的时间 |

## 使用FlinkSQL通过CDC接入Doris示例

```sql
-- enable checkpoint  
SET 'execution.checkpointing.interval' = '10s';  
  
CREATE TABLE cdc_mysql_source (  
id int  
,name VARCHAR  
,PRIMARY KEY (id) NOT ENFORCED  
) WITH (  
'connector' = 'mysql-cdc',  
'hostname' = '127.0.0.1',  
'port' = '3306',  
'username' = 'root',  
'password' = 'password',  
'database-name' = 'database',  
'table-name' = 'table'  
);  
  
-- 支持同步insert/update/delete事件  
CREATE TABLE doris_sink (  
id INT,  
name STRING  
)  
WITH (  
'connector' = 'doris',  
'fenodes' = '127.0.0.1:8030',  
'table.identifier' = 'database.table',  
'username' = 'root',  
'password' = '',  
'sink.properties.format' = 'json',  
'sink.properties.read_json_by_line' = 'true',  
'sink.enable-delete' = 'true', -- 同步删除事件  
'sink.label-prefix' = 'doris_label'  
);  
  
insert into doris_sink select id,name from cdc_mysql_source;
```


## 使用FlinkCDC接入多表或整库示例

### 语法

```shell
<FLINK_HOME>bin/flink run \
    -c org.apache.doris.flink.tools.cdc.CdcTools \
    lib/flink-doris-connector-1.16-1.4.0-SNAPSHOT.jar \
    <mysql-sync-database|oracle-sync-database|postgres-sync-database|sqlserver-sync-database> \
    --database <doris-database-name> \
    [--job-name <flink-job-name>] \
    [--table-prefix <doris-table-prefix>] \
    [--table-suffix <doris-table-suffix>] \
    [--including-tables <mysql-table-name|name-regular-expr>] \
    [--excluding-tables <mysql-table-name|name-regular-expr>] \
    --mysql-conf <mysql-cdc-source-conf> [--mysql-conf <mysql-cdc-source-conf> ...] \
    --oracle-conf <oracle-cdc-source-conf> [--oracle-conf <oracle-cdc-source-conf> ...] \
    --sink-conf <doris-sink-conf> [--table-conf <doris-sink-conf> ...] \
    [--table-conf <doris-table-conf> [--table-conf <doris-table-conf> ...]]
```

### 样例

```shell
mysql-sync-database   
--database ODS 
--table-prefix test_  
--mysql-conf hostname=${test.host}  
--mysql-conf port=3306  
--mysql-conf username=${test.user}  
--mysql-conf password=${test.password}  
--mysql-conf database-name=test 
--mysql-conf scan.newly-added-table.enabled=TRUE 
--mysql-conf jdbc.properties.useSSL=false 
--mysql-conf jdbc.properties.tinyInt1isBit=false 
--including-tables TEST.* 
--excluding-tables TEST_PRICE|TEST_TEMP|
--sink-conf fenodes=${doris.fenodes}  
--sink-conf username=${doris.user}  
--sink-conf password=${doris.password}  
--sink-conf jdbc-url=${doris.url} 
--sink-conf sink.enable-2pc=FALSE 
--use-new-schema-change 
--table-conf replication_num=1
```

配置 

- **--job-name** Flink任务名称, 非必需。
- **--database** 同步到Doris的数据库名。
- **--table-prefix** Doris表前缀名，例如 --table-prefix ods_。
- **--table-suffix** 同上，Doris表的后缀名。
- **--including-tables** 需要同步的MySQL表，可以使用"|" 分隔多个表，并支持正则表达式。 比如--including-tables table1|tbl.*就是同步table1和所有以tbl开头的表。
- **--excluding-tables** 不需要同步的表，用法同上。
- **--mysql-conf** MySQL CDCSource 配置，例如--mysql-conf hostname=127.0.0.1 ，您可以在 [[MySQL-CDC-Connector(v2.4)#^2fd7d6|mysql-cdc配置]] 查看所有配置MySQL-CDC，其中hostname/username/password/database-name 是必需的。
- **--oracle-conf** Oracle CDCSource 配置，例如--oracle-conf hostname=127.0.0.1 ，您可以在[这里](https://ververica.github.io/flink-cdc-connectors/master/content/connectors/oracle-cdc.html)查看所有配置Oracle-CDC，其中hostname/username/password/database-name/schema-name 是必需的。
- **--sink-conf** Doris Sink 的所有配置，可以在[这里](https://doris.apache.org/zh-CN/docs/dev/ecosystem/flink-doris-connector/#%E9%80%9A%E7%94%A8%E9%85%8D%E7%BD%AE%E9%A1%B9)   [[#^2f987c|配置]]  查看完整的配置项。
- **--table-conf** Doris表的配置项，即properties中包含的内容。 例如 --table-conf replication_num=1
- **--ignore-default-value** 关闭同步mysql表结构的默认值。适用于同步mysql数据到doris时，字段有默认值，但实际插入数据为null情况。参考[#152](https://github.com/apache/doris-flink-connector/pull/152)
- **--use-new-schema-change** 新的schema change支持同步mysql多列变更、默认值。参考[#167](https://github.com/apache/doris-flink-connector/pull/167)


## 使用FlinkCDC更新Key列

一般在业务数据库中，会使用编号来作为表的主键，比如Student表，会使用编号(id)来作为主键，但是随着业务的发展，数据对应的编号有可能是会发生变化的。 在这种场景下，使用FlinkCDC + Doris Connector同步数据，便可以自动更新Doris主键列的数据。

### 原理

Flink CDC底层的采集工具是Debezium，Debezium内部使用op字段来标识对应的操作：op字段的取值分别为c、u、d、r，分别对应create、update、delete和read。 而对于主键列的更新，FlinkCDC会向下游发送DELETE和INSERT事件，同时数据同步到Doris中后，就会自动更新主键列的数据。

### 使用

Flink程序可参考上面CDC同步的示例，成功提交任务后，在MySQL侧执行Update主键列的语句(`update student set id = '1002' where id = '1001'`)，即可修改Doris中的数据。


## 使用Flink根据指定列删除数据

一般Kafka中的消息会使用特定字段来标记操作类型，比如`{"op_type":"delete",data:{...}}`。
针对这类数据，希望将op_type=delete的数据删除掉。

DorisSink默认会根据RowKind来区分事件的类型，通常这种在cdc情况下可以直接获取到事件类型，对隐藏列`__DORIS_DELETE_SIGN__`进行赋值达到删除的目的，而Kafka则需要根据业务逻辑判断，显示的传入隐藏列的值。

### 使用

```sql
-- 比如上游数据: {"op_type":"delete",data:{"id":1,"name":"zhangsan"}}
CREATE TABLE KAFKA_SOURCE(
  data STRING,
  op_type STRING
) WITH (
  'connector' = 'kafka',
  ...
);

CREATE TABLE DORIS_SINK(
  id INT,
  name STRING,
  __DORIS_DELETE_SIGN__ INT
) WITH (
  'connector' = 'doris',
  'fenodes' = '127.0.0.1:8030',
  'table.identifier' = 'db.table',
  'username' = 'root',
  'password' = '',
  'sink.enable-delete' = 'false',        -- false表示不从RowKind获取事件类型
  'sink.properties.columns' = 'id, name, __DORIS_DELETE_SIGN__'  -- 显示指定streamload的导入列
);

INSERT INTO DORIS_SINK
SELECT json_value(data,'$.id') as id,
json_value(data,'$.name') as name, 
if(op_type='delete',1,0) as __DORIS_DELETE_SIGN__ 
from KAFKA_SOURCE;
```


## Java示例

`samples/doris-demo/` 下提供了 Java 版本的示例，可供参考，查看点击[这里](https://github.com/apache/doris/tree/master/samples/doris-demo/)


## 最佳实践

### 应用场景

使用 Flink Doris Connector最适合的场景就是实时/批次同步源数据（Mysql，Oracle，PostgreSQL等）到Doris，使用Flink对Doris中的数据和其他数据源进行联合分析，也可以使用Flink Doris Connector。

### 其他

1. Flink Doris Connector主要是依赖Checkpoint进行流式写入，所以Checkpoint的间隔即为数据的可见延迟时间。
2. 为了保证Flink的Exactly Once语义，Flink Doris Connector 默认开启两阶段提交，Doris在1.1版本后默认开启两阶段提交。1.0可通过修改BE参数开启，可参考[two_phase_commit](https://doris.incubator.apache.org/zh-CN/docs/data-operate/import/import-way/stream-load-manual)。