---
title: hive2.1.1 无法识别高版本hive-orc数据
author: xxzuo
tags:
  - hive
categories:
  - 大数据
date: 2023-07-10 00:49:20
---

在 利用 distcp 将 原 apahce hadoop 集群 传输数据到 新 cdh6.3.2 集群后
在 新 cdh 集群上 使用 hive 建表，结果 无法查询到 表数据提示 

> ORC split generation failed with exception:ArrayIndexOutOfBoundsException: 7

查询资料发现
https://issues.apache.org/jira/browse/HIVE-14483

这是由于 HIVE 在 写 ORC 格式数据时，会附带一个 `VERSION`
在 READ 数据的时候 会获取这个 `VERSION`
但是在 2.1.1 版本的 代码中
WriterVersion 这个 枚举类的`from`方法

![[orc_error1.png]]

这个 返回的 枚举类型 有问题
在高版本情况下 ,如果 val  =  7 时
它不会走到 
`return FUTURE;`
会 
`return values[7];`
这里很明显就会导致数据越界问题所以我们需要修改这里的代码
改成如下代码:

![[orc_error2.png]]

做一个判断如果 val 大于等于values的长度，说明是一个高版本的数据，直接返回 FUTURE
重新编译class文件以后，更新 jar包
这里涉及到  `hive-exec`  和 `hive-orc` 两个jar包
更新完 jar 以后 上传到 服务器，重启 hive 即可