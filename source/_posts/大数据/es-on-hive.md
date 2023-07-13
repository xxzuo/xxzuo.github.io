---
title: es on hive
tags:
  - hive
  - ElasticSearch
categories:
  - 大数据
date: 2022-06-16 21:50:59
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
    column1     string,     
    column2  string,     
    column3 int
)    stored by 'org.elasticsearch.hadoop.hive.EsStorageHandler'    
tblproperties (         
    'es.resource' = 'chat/_doc',         
    'es.nodes' = '192.168.x.x',        
    "es.nodes.wan.only" = "true",         
    'es.transport.port' = '9200',         
    'es.mapping.names' =                
    'column1:column1 ,column2:column2, column3:column3'         
);
```



4.取出es数据

对hive表进行 

insert overwrite table xxx 

select * from xxx;即可取出数据

hive表配置

tblproperties

参考[Configuration | Elasticsearch for Apache Hadoop [7.13\] | Elastic](https://www.elastic.co/guide/en/elasticsearch/hadoop/current/configuration.html)
