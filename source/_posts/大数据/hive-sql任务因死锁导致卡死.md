---
title: hive锁
author: xxzuo
categories:
  - 大数据
date: 2024-02-02 13:56:48
tags:
---

# 起因

某次 有一个 HIVE SQL 任务 突然报错
报错如下
```
 nested exception is java.sql.SQLException: Error while processing statement: FAILED: Error in acquiring locks: Locks on the underlying objects cannot be acquired. retry after some time
        at org.springframework.jdbc.support.SQLStateSQLExceptionTranslator.doTranslate(SQLStateSQLExceptionTranslator.java:101)
        at org.springframework.jdbc.support.AbstractFallbackSQLExceptionTranslator.translate(AbstractFallbackSQLExceptionTranslator.java:72)
        at org.springframework.jdbc.support.AbstractFallbackSQLExceptionTranslator.translate(AbstractFallbackSQLExceptionTranslator.java:81)
        at org.springframework.jdbc.support.AbstractFallbackSQLExceptionTranslator.translate(AbstractFallbackSQLExceptionTranslator.java:81)
        at 

```


# 问题定位

使用 `show locks`  发现 某张表 存在 `SHARED` 锁

再通过 `show locks extended`  查询 锁的具体情况



# 解锁

- 直接通过 命令 `unlock table table_name` 解锁， 重新执行 SQL 任务即可

- 设置 hive 参数 `set hive.support.concurrency=false` 来不用 hive 的并发锁机制

