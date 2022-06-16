---
title: es on hive
date: 2022-06-16 21:50:59
tags:
- hive
categories: 
- 配置相关
---



# es on hive



参考https://www.elastic.co/guide/en/elasticsearch/hadoop/current/hive.html



1.首先在官网下载jar包，解压到hive/lib目录下 [Download Elasticsearch for Hadoop Free | Elastic](https://www.elastic.co/cn/downloads/hadoop) 

2.hive server执行时，添加参数 

> hive.aux.jars.path=/path/elasticsearch-hadoop.jar 

或者修改 hive-site.xml 添加

```xml
<property>
<name>hive.aux.jars.path</name>
<value>/path/elasticsearch-hadoop.jar</value>
<description>A comma separated list (with no spaces) of the jar files</description>
</property>
```





3.创建hive表，用于映射 es

```sql
create external table test.es_msg 
(     
    chatType     string,     
    fromAccount  string,     
    msgTimestamp bigint,     
    toAccount    string 
)    stored by 'org.elasticsearch.hadoop.hive.EsStorageHandler'    
tblproperties (         
    'es.resource' = 'chat1/_doc',         
    'es.nodes' = '192.168.2.146',        
    "es.nodes.wan.only" = "true",         
    'es.transport.port' = '9200',         
    'es.mapping.names' =                
    'chatType:chatType ,fromAccount:fromAccount, msgTimestamp:msgTimestamp, toAccount:toAccount'         
);
```



4.取出es数据

对hive表进行 

insert overwrite table xxx 

select * from xxx;即可取出数据

hive表配置

tblproperties

参考[Configuration | Elasticsearch for Apache Hadoop [7.13\] | Elastic](https://www.elastic.co/guide/en/elasticsearch/hadoop/current/configuration.html)
