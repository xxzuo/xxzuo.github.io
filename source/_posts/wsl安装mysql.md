---
title: wsl安装mysql
author: xxzuo
date: 2022-06-25 00:40:42
tags:
- mysql
categories:
- 配置相关
---



```shell
# 先查看是否有安装过的 mysql
dpkg -l | grep mysql

# 如果没有 安装 mysql-server
sudo apt install mysql-server

# 等待安装完成后 启动 mysql 
sudo service mysql start


# 利用 net-tools 查看mysql 是否启动
netstat -tap | grep mysql

```



```shell
# 登录mysql
mysql -u root -p
```

此时提示

> Enter password:
> ERROR 2002 (HY000): Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock' (13)

发现我们不知道 mysql 的初始密码

此时 需要查看 mysql 生成的默认账号

```shell
sudo cat /etc/mysql/debian.cnf
```

结果如下

> # Automatically generated for Debian scripts. DO NOT TOUCH!
> [client]
> host     = localhost
> user     = debian-sys-maint
> password = heuEWrIzKnl6beJQ
> socket   = /var/run/mysqld/mysqld.sock
> [mysql_upgrade]
> host     = localhost
> user     = debian-sys-maint
> password = heuEWrIzKnl6beJQ
> socket   = /var/run/mysqld/mysqld.sock



此处 记录了 mysql 自动生成的账号和密码

利用自带的账号登录即可



>  Can't connect to local MySQL server through socket '/var/run/mysqld/mysqld.sock' (13)



