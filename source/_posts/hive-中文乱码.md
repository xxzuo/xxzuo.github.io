---
title: hive 中文乱码
date: 2022-06-16 21:51:19
tags:
- hive
categories: 
- 大数据
---



# hive 中文乱码



## 修改元数据库字符集

```sql
-- 修改表字段 注解 和 表 注解
ALTER TABLE COLUMNS_V2 MODIFY COLUMN COMMENT varchar(256) character set utf8 ;
ALTER TABLE TABLE_PARAMS MODIFY COLUMN PARAM_VALUE varchar(4000) character set utf8 ;
-- 修改分区字段注解
ALTER TABLE PARTITION_PARAMS MODIFY COLUMN PARAM_VALUE varchar(4000) character set utf8 ;
ALTER TABLE PARTITION_KEYS MODIFY COLUMN PKEY_COMMENT varchar(4000) character set utf8 ;
-- 修改索引注解
ALTER TABLE INDEX_PARAMS MODIFY COLUMN PARAM_VALUE varchar(4000) character set utf8 ;
-- 修改 数据库 注解
ALTER TABLE DBS MODIFY COLUMN `DESC` varchar(4000) character set utf8 ;
```

