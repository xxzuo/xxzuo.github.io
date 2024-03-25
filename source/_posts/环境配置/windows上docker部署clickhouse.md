---
title: windows上docker部署clickhouse
author: xxzuo
tags:
  - clickhouse
  - docker
categories:
  - 环境配置
date: 2024-03-25 22:17:35
---

## 一、拉取镜像

```shell
docker pull yandex/clickhouse-server:21.3.20.1
```

> 注: 不知道为什么   latest 版本有问题，无法启动， 所以这里指定了  21.3.20.1

## 二、运行临时容器

这一步主要是为了获取 配置文件 `config.xml` 和 `users.xml`
```shell
docker run --rm -d --name=temp-clickhouse-server yandex/clickhouse-server:21.3.20.1
```


## 三、创建windows本地目录

这一步 主要是 为了映射

> 创建目录 `E:\bigdata\clickhouse`
> 创建目录 `E:\bigdata\clickhouse\config`
> 创建目录 `E:\bigdata\clickhouse\data`
> 创建目录 `E:\bigdata\clickhouse\logs`

复制文件
```shell
docker cp temp-clickhouse-server:/etc/clickhouse-server/users.xml E:/bigdata/clickhouse/conf/users.xml

 docker cp temp-clickhouse-server:/etc/clickhouse-server/config.xml E:/bigdata/clickhouse/conf/config.xml
```
   
## 四、创建账号

进入容器
```shell
docker exec -it temp-clickhouse-server /bin/bash
```

获取 sha256 加密字符串

```shell
PASSWORD=$(base64 < /dev/urandom | head -c8); echo "test"; echo -n "123456" | sha256sum | tr -d '-'
```

这里会得到一个结果
```
test
8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92
```
也可以考虑 使用 在线网站 比如  [SHA256 在线加密工具 | 菜鸟工具 (jyshare.com)](https://www.jyshare.com/crypto/sha256/) 生成


然后编辑 `users.xml` 文件
在 `<users>` 标签下面添加
```xml
<test>
    <password_sha256_hex>8d969eef6ecad3c29a3a629280e686cf0c3f5d5a86aff3ca12020c923adc6c92</password_sha256_hex>
	<networks incl="networks" replace="replace">
    <ip>::/0</ip>
    </networks>
    <profile>default</profile>
    <quota>default</quota>
</test>
```


修改 `config.xml` 文件
添加下面这行
```xml
<listen_host>0.0.0.0</listen_host>
```

销毁上面启动的临时容器
```
docker stop temp-clickhouse-server
```


## 五、正式启动

```shell
docker run -d --name=single-clickhouse-server -p 8123:8123 -p 9000:9000 -p 9009:9009 --ulimit nofile=262144:262144 --volume E:/bigdata/clickhouse/data:/var/lib/clickhouse:rw --volume E:/bigdata/clickhouse/conf/config.xml:/etc/clickhouse-server/config.xml --volume E:/bigdata/clickhouse/conf/users.xml:/etc/clickhouse-server/users.xml --volume E:/bigdata/clickhouse/logs:/var/log/clickhouse-server:rw yandex/clickhouse-server:21.3.20.1
```

