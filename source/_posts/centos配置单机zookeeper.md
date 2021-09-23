---
title: centos配置单机zookeeper
date: 2021-05-07 22:12:40
categories: 
- 配置相关
tags:
- centos
- zookeeper
- config
---



### 1.下载zookeeper

[下载地址](https://mirrors.bfsu.edu.cn/apache/zookeeper/zookeeper-3.6.3/apache-zookeeper-3.6.3-bin.tar.gz)

2.上传到服务器

```shell
# 进入上传zookeeper的目录
cd /usr/local
# 解压
tar -zxvf apache-zookeeper-3.6.3-bin.tar.gz
# 进入zookeeper解压目录
cd apache-zookeeper-3.6.3-bin/
# 新建data文件夹，路径根据个人习惯
mkdir data
# 新建日志文件夹，路径根据个人习惯
mkdir logs
# 进入配置文件夹
cd conf
# 复制给的模板配置文件
cp zoo_sample.cfg zoo.cfg
# 修改给配置文件 zoo.cfg
vi zoo.cfg
```

主要关注

```shell
#三个属性
# 数据目录 刚才新建的data
dataDir
# 日志目录 刚才新建的logs
dataLogDir
# 端口
clientPort 

```

然后执行

```she
sh zkServer.sh start
```

