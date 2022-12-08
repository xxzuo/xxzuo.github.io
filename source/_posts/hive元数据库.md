---
title: hive元数据库
author: xxzuo
date: 2022-07-28 22:18:58
tags:
- hive
categories:
- 大数据
hide: true
---



# hive元数据库

## 配置
查看 hive-site.xml 内容

```xml
  <property>
    <name>javax.jdo.option.ConnectionURL</name>
    <value>jdbc:mysql://localhost:3306/hive_meta?useUnicode=true&amp;characterEncoding=UTF-8&amp;autoReconnect=true&amp;maxReconnects=10&amp;useSSL=false</value>
  </property>

  <property>
    <name>javax.jdo.option.ConnectionDriverName</name>
    <value>com.mysql.jdbc.Driver</value>
  </property>

  <property>
    <name>javax.jdo.option.ConnectionUserName</name>
    <value>username</value>
  </property>

  <property>
    <name>javax.jdo.option.ConnectionPassword</name>
    <value>password</value>
  </property>
```


## hive元数据表信息

| 表名                                | 注释       |
| ----------------------------------- | ---------- |
| AUX_TABLE                           |            |
| BUCKETING_COLS                      |            |
| CDS                                 |            |
| COLUMNS_V2                          | 字段信息   |
| COMPACTION_QUEUE                    |            |
| COMPLETED_COMPACTIONS               |            |
| COMPLETED_TXN_COMPONENTS            |            |
| CTLGS                               |            |
| [DATABASE_PARAMS](#database_params) | 数据库参数 |
| [DBS](#dbs)                         | 数据库信息 |
| DB_PRIVS                            |            |
| DELEGATION_TOKENS                   |            |
| FUNCS                               |            |
| FUNC_RU                             |            |
| GLOBAL_PRIVS                        |            |
| HIVE_LOCKS                          |            |
| IDXS                                |            |
| INDEX_PARAMS                        |            |
| I_SCHEMA                            |            |
| KEY_CONSTRAINTS                     |            |
| MASTER_KEYS                         |            |
| MATERIALIZATION_REBUILD_LOCKS       |            |
| METASTORE_DB_PROPERTIES             |            |
| MIN_HISTORY_LEVEL                   |            |
| MV_CREATION_METADATA                |            |
| MV_TABLES_USED                      |            |
| NEXT_COMPACTION_QUEUE_ID            |            |
| NEXT_LOCK_ID                        |            |
| NEXT_TXN_ID                         |            |
| NEXT_WRITE_ID                       |            |
| NOTIFICATION_LOG                    |            |
| NOTIFICATION_SEQUENCE               |            |
| NUCLEUS_TABLES                      |            |
| PARTITIONS                          |            |
| PARTITION_EVENTS                    |            |
| PARTITION_KEYS                      |            |
| PARTITION_KEY_VALS                  |            |
| PARTITION_PARAMS                    |            |
| PART_COL_PRIVS                      |            |
| PART_COL_STATS                      |            |
| PART_PRIVS                          |            |
| REPL_TXN_MAP                        |            |
| ROLES                               |            |
| ROLE_MAP                            |            |
| RUNTIME_STATS                       |            |
| SCHEMA_VERSION                      |            |
| SDS                                 |            |
| SD_PARAMS                           |            |
| SEQUENCE_TABLE                      |            |
| SERDES                              |            |
| SERDE_PARAMS                        |            |
| SKEWED_COL_NAMES                    |            |
| SKEWED_COL_VALUE_LOC_MAP            |            |
| SKEWED_STRING_LIST                  |            |
| SKEWED_STRING_LIST_VALUES           |            |
| SKEWED_VALUES                       |            |
| SORT_COLS                           |            |
| TABLE_PARAMS                        |            |
| TAB_COL_STATS                       |            |
| TBLS                                | 表信息     |
| TBL_COL_PRIVS                       |            |
| [TBL_PRIVS](#tbl_privs)             | 表权限信息 |
| TXNS                                |            |
| TXN_COMPONENTS                      |            |
| TXN_TO_WRITE_ID                     |            |
| TYPES                               |            |
| TYPE_FIELDS                         |            |
| [VERSION](#version)                 | hive版本   |
| WM_MAPPING                          |            |
| WM_POOL                             |            |
| WM_POOL_TO_TRIGGER                  |            |
| WM_RESOURCEPLAN                     |            |
| WM_TRIGGER                          |            |
| WRITE_SET                           |            |


## hive元数据表详情

### <span id='version'>VERSION(存储Hive版本的元数据表)</span>

| 字段名          | 注释     |
| --------------- | -------- |
| VER_ID          | 主键     |
| SCHEMA_VERSION  | Hive版本 |
| VERSION_COMMENT | 版本说明 |


### <span id='dbs'>DBS(存储数据库信息)</span>

| 字段名          | 注释             |
| --------------- | ---------------- |
| DB_ID           | 主键             |
| DESC            | 数据库描述       |
| DB_LOCATION_URI | hdfs路径         |
| NAME            | 数据库名称       |
| OWNER_NAME      | 数据库所有者名称 |
| OWNER_TYPE      | 数据库所有者角色 |
| CTLG_NAME       | catalog名称      |



### <span id='database_params'>DATABASE_PARAMS(存储数据库参数信息)</span>

| 字段名      | 注释     |
| ----------- | -------- |
| DB_ID       | 数据库ID |
| PARAM_KEY   | 参数名   |
| PARAM_VALUE | 参数值   |



### <span id='db_privs'>DB_PRIVS(数据库权限信息)</span>

| 字段名         | 注释     |
| -------------- | -------- |
| DB_GRANT_ID    | 主键ID   |
| CREATE_TIME    | 创建时间 |
| DB_ID          | 数据库ID |
| GRANT_OPTION   |          |
| GRANTOR        |          |
| GRANTOR_TYPE   |          |
| PRINCIPAL_NAME |          |
| PRINCIPAL_TYPE |          |
| DB_PRIV        |          |
| AUTHORIZER     |          |







### <span id='tbl_privs'>TBL_PRIVS(表权限)</span>

| 字段名 | 注释 |
| ------ | ---- |
|        |      |
|        |      |
|        |      |
