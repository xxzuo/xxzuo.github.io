---
title: hive权限相关命令
author: xxzuo
date: 2022-06-22 23:27:58
tags:
- hive
categories:
- 大数据
---



```sql
-- 查看当前用户
select current_user();
 
 
-- 查看当前角色
show current roles;
 
 
-- 切换 admin 角色
set role admin;
 
-- 查看所有角色
show roles;
 
 
 
--  查看用户的角色
show role grant user {userName};
 
 
 
 
-- 把角色授权给某个用户
grant role {roleName} to user {userName};
 
 
 
 
-- 撤销某个用户的角色授权
revoke role {roleName} from user {userName};
 
 
 
 
 
 
-- 创建角色
create role {roleName};
 
 
 
 
-- 删除角色
drop role {roleName};
 
 
-- 授予某个库的权限给某个用户
grant select on database {dbName} to user {userName};
grant insert on database {dbName} to user {userName};
grant update on database {dbName} to user {userName};
grant delete on database {dbName} to user {userName};
 
-- 回收某个库的权限给某个用户
revoke select on database {dbName} from user {userName};
revoke insert on database {dbName} from user {userName};
revoke update on database {dbName} from user {userName};
revoke delete on database {dbName} from user {userName};
 
-- 查看指定用户在所有库下面的权限
show grant user {userName};
-- 查看指定用户在某个库的权限
show grant user {userName} on database {dbName};
 
 
 
 
 
 
-- 授予表的权限给某个用户
grant select on table {dbName}.tableName to user {userName};
grant insert on table {dbName}.tableName to user {userName};
grant update on table {dbName}.tableName to user {userName};
grant delete on table {dbName}.tableName to user {userName};
 
 
-- 回收某个用户的表的权限
revoke create on table {dbName}.tableName from user {userName};
revoke select on table {dbName}.tableName from user {userName};
revoke insert on table {dbName}.tableName from user {userName};
revoke update on table {dbName}.tableName from user {userName};
revoke delete on table {dbName}.tableName from user {userName};
 
-- 查看指定用户在指定表的权限
show grant user {userName} on table {dbName}.{tableName};
 
 
 
 
 
-- 权限类别
-- ALTER  更改表结构，创建分区
-- CREATE  创建表
-- DROP  删除表，或分区
-- INDEX  创建和删除索引
-- LOCK  锁定表，保证并发
-- SELECT  查询表权限
-- SHOW_DATABASE  查看数据库权限
-- UPDATE  为表加载本地数据的权限
```
