---
title: centos7-hadoop单机配置
date: 2021-05-03 12:35:35
hide: true
tags:
- centos
- hadoop
categories: 
- 配置相关
---



准备搭一个hadoop单机环境用于学习hdfs和hive

## 1.配置java

参考[centos配置java - Hexo (xxzuo.github.io)](https://xxzuo.github.io/2021/05/03/centos配置java/)



## 2.hadoop下载

下载地址：[Apache Download Mirrors](https://www.apache.org/dyn/closer.cgi/hadoop/common/hadoop-3.2.2/hadoop-3.2.2.tar.gz)



### 3.上传到服务器并解压

```shell
# 解压路径根据个人习惯
tar -zxvf hadoop-3.2.2.tar.gz 
```



## 4.配置hadoop环境变量

编辑配置文件

```shell
vi /etc/profile
```

在文件底部增加

```shell
export HADOOP_HOME=/home/app/hadoop/hadoop-3.2.2
export PATH=$PATH:$HADOOP_HOME/bin
```

刷新配置

```shell
 source /etc/profile
```

查看hadoop 版本

```shell
 hadoop version
```



## 5.hadoop配置

### 5.1配置hadoop-env.sh

编辑hadoop-env.sh

```shell
vi /home/app/hadoop/hadoop-3.2.2/etc/hadoop/hadoop-env.sh 
```

找到 # The java implementation to use. By default, this environment 所在位置(可以使用vim的正则匹配寻找)

更改下面的配置为

```shell
export JAVA_HOME=/usr/local/java/jdk1.8.0_202
```

如图所示：

<img src="/img/hadoop/1.jpg">

### 5.2配置core-site.xml

编辑core-site.xml

```shell
vi /home/app/hadoop/hadoop-3.2.2/etc/hadoop/core-site.xml
```

```xml
<configuration>
        <property>
                <name>hadoop.tmp.dir</name>
                <value>/home/app/hadoop/hadoop-3.2.2/tmp</value>
                <description>Abase for other temporary directories.</description>
        </property>
        <property>
                <name>fs.defaultFS</name>
                <value>hdfs://ip:9000</value>
        </property>
</configuration>

```





### 5.3配置hdfs-site.xml

编辑hdfs-site.xml

```shell
 vi /home/app/hadoop/hadoop-3.2.2/etc/hadoop/hdfs-site.xml
```



```xml
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/home/app/hadoop/hadoop-3.2.2/hdfs/name</value>
    </property>
    <property>
        <name>dfs.namenode.data.dir</name>
        <value>/home/app/hadoop/hadoop-3.2.2/hdfs/data</value>
    </property>
</configuration>
```





### 6.hadoop用户

新建hadoop用户

```shell
useradd -m hadoop -s /bin/bash
```

设置密码

```shell
passwd hadoop
```

此时 如果输入的密码比较简单会提示

`无效的密码： 密码少于 8 个字符`忽略即可

### 7.单机版免密登录

切换到hadoop用户登录

```shell
su hadoop
```

测试ssh

```shell
ssh localhost
```

此时需要输入密码。输入hadoop用户密码成功之后。



然后我们输入如下命令

```shell
cd ~/.ssh/
ssh-keygen -t rsa
# 此时会有提示，一直按回车即可

# 加入授权
cat id_rsa.pub >> authorized_keys  
# 修改文件权限
chmod 600 ./authorized_keys    
```

再次测试ssh

```shell
ssh localhost
```

发现不需要密码登录了

<img src="/img/hadoop/2.jpg">



### 8.启动hadoop

在hadoop用户下启动hadoop

进入hadoop目录

```shell
cd /home/app/hadoop/hadoop-3.2.2/
```

启动前先初始化

```shell
./bin/hdfs namenode -format
```

会有提示询问，输入Y即可

启动

```shell
./sbin/start-dfs.sh
```

可能出现的错误:

1.如果出现`Warning: Permanently added`错误时

创建~/.ssh/config文件

```shell
vi ~/.ssh/config
```

在文件中输入

```shell
UserKnownHostsFile ~/.ssh/known_hosts
```



2.如果出现`SSH: Bad owner or permissions on .ssh/config`错误

采用https://blog.csdn.net/zcc_heu/article/details/79017606

中的解决方法

```shell
sudo chmod 600 ~/.ssh/config
```



