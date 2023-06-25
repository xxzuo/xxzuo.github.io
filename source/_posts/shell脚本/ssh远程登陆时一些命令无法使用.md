---
title: ssh远程登陆时一些命令无法使用
author: xxzuo
categories:
  - shell脚本
date: 2022-07-22 21:53:26
tags:
---



在180 ssh 到另一台服务器179 执行命令时报错
```bash
[root@180 sbin]# ssh 179 -p 60022 "mapred --help"
bash: mapred: 未找到命令
```
但实际在179 可以执行该命令
```bash
[root@179 sbin]# mapred --help
Usage: mapred [OPTIONS] SUBCOMMAND [SUBCOMMAND OPTIONS]
or mapred [OPTIONS] CLASSNAME [CLASSNAME OPTIONS]
where CLASSNAME is a user-provided Java class

OPTIONS is none or any of:

--config dir Hadoop config directory
--debug turn on shell script debug mode
--help usage information

SUBCOMMAND is one of:


Admin Commands:

frameworkuploader mapreduce framework upload
hsadmin job history server admin interface

Client Commands:

classpath prints the class path needed for running mapreduce subcommands
envvars display computed Hadoop environment variables
job manipulate MapReduce jobs
minicluster CLI MiniCluster
pipes run a Pipes job
queue get information regarding JobQueues
sampler sampler
version print the version

Daemon Commands:

historyserver run job history servers as a standalone daemon

SUBCOMMAND may print help when invoked w/o parameters or with -h.
```


这是 ssh 在登录远程的环境变量中不包含可执行文件的路径, 所以要自己加上路径

```bash
[root@180 sbin]# ssh 179 -p 60022 "/usr/local/hadoop/hadoop-3.2.2/bin/mapred --help"
Usage: mapred [OPTIONS] SUBCOMMAND [SUBCOMMAND OPTIONS]
or mapred [OPTIONS] CLASSNAME [CLASSNAME OPTIONS]
where CLASSNAME is a user-provided Java class

OPTIONS is none or any of:

tput: No value for $TERM and no -T specified
--config dir Hadoop config directory
--debug turn on shell script debug mode
--help usage information

SUBCOMMAND is one of:


Admin Commands:

tput: No value for $TERM and no -T specified
frameworkuploader mapreduce framework upload
hsadmin job history server admin interface

Client Commands:

tput: No value for $TERM and no -T specified
classpath prints the class path needed for running mapreduce
subcommands
envvars display computed Hadoop environment variables
job manipulate MapReduce jobs
minicluster CLI MiniCluster
pipes run a Pipes job
queue get information regarding JobQueues
sampler sampler
version print the version

Daemon Commands:

tput: No value for $TERM and no -T specified
historyserver run job history servers as a standalone daemon

SUBCOMMAND may print help when invoked w/o parameters or with -h.
```


这样就可以执行了
