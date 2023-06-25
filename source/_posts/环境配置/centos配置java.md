---
title: centos配置java
categories:
  - 环境配置
tags:
  - centos
  - java
  - config
date: 2021-05-03 11:30:43
---

## tar.gz包安装

#### 1.jdk下载

关于jdk历史版本的下载，可以在这里找到[Releases · frekele/oracle-java (github.com)](https://github.com/frekele/oracle-java/releases)

<img src="/img/centos_config_java/1.jpg">

我们选择[` jdk-8u202-linux-x64.tar.gz`](https://github.com/frekele/oracle-java/releases/download/8u202-b08/jdk-8u202-linux-x64.tar.gz)

2.服务器配置

```shell
# 在服务器上新建目录
cd /usr/local
mkdir java
```

利用[finalshell]([SSH工具 客户端 (hostbuf.com)](http://www.hostbuf.com/)) 将 `jdk-8u202-linux-x64.tar.gz` 上传到该目录

```shell
# 解压
tar -zxvf jdk-8u202-linux-x64.tar.gz 

# 配置环境变量
vi /etc/profile

# 在文件底部 输入
export JAVA_HOME=/usr/local/java/jdk1.8.0_202
export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
export PATH=$JAVA_HOME/bin:$PATH

# 刷新配置
source /etc/profile

```

<img src="/img/centos_config_java/2.jpg">

配置好后，输入如下命令

```shell
java -version
```

出现

<img src="/img/centos_config_java/3.jpg">

就表示安装成功



