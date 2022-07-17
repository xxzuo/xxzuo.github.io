---
title: flink无法用stop-cluster.sh停止
author: xxzuo
date: 2022-07-16 22:21:31
tags:
- flink
categories:
- 配置相关
---



使用 [stop-cluster.sh](http://stop-cluster.sh/) 关闭 flink 时失败

```bash
./stop-cluster.sh
# No taskexecutor daemon to stop on host beideng-nj-prod-hadoop-02.
# No standalonesession daemon to stop on host beideng-nj-prod-hadoop-02.
```

但是 ps -ef|grep flink 可以看到 flink 进程
访问flink web 界面也正常

和 hadoop 一样，flink 在启动的时候会将 PID 存放到一个目录中。默认是 /tmp 目录
由于 /tmp 目录会定期删除，所以找不到 PID 文件，集群停止失败

更改 PID 目录即可

```bash
vi flink/bin/config.sh

# WARNING !!! , these values are only used if there is nothing else is specified in
# conf/flink-conf.yaml

DEFAULT_ENV_PID_DIR="/tmp"                          # Directory to store *.pid files to
DEFAULT_ENV_LOG_MAX=10                              # Maximum number of old log files to keep
DEFAULT_ENV_JAVA_OPTS=""                            # Optional JVM args
DEFAULT_ENV_JAVA_OPTS_JM=""                         # Optional JVM args (JobManager)
DEFAULT_ENV_JAVA_OPTS_TM=""                         # Optional JVM args (TaskManager)
DEFAULT_ENV_JAVA_OPTS_HS=""                         # Optional JVM args (HistoryServer)
DEFAULT_ENV_JAVA_OPTS_CLI=""                        # Optional JVM args (Client)
DEFAULT_ENV_SSH_OPTS=""                             # Optional SSH parameters running in cluster mode
DEFAULT_YARN_CONF_DIR=""                            # YARN Configuration Directory, if necessary
DEFAULT_HADOOP_CONF_DIR=""                          # Hadoop Configuration Directory, if necessary
DEFAULT_HBASE_CONF_DIR=""                           # HBase Configuration Directory, if necessary

 # 修改 DEFAULT_ENV_PID_DIR="/tmp" 地址
 # 改为自己指定的地址 如 DEFAULT_ENV_PID_DIR=“/var/run/flink/$USER”
 # 再次启动 flink
 ./start-cluster.sh
 
 # 进入刚才修改的目录
 cd /var/run/flink/$USER
 
 # 发现已经生成 PID 文件
 ls
 # flink-root-standalonesession.pid
 # flink-root-taskexecutor.pid
```
