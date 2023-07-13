---
title: flume启动脚本
author: xxzuo
tags:
  - flume
categories:
  - flume
date: 2022-12-08 23:13:37
---

## 自定义flume启停脚本

使用 shell 编写一个 flume 启动、停止、重启脚本

执行 方式 ：

sh 脚本 (start|stop|restart)

**注意： 需要将该 脚本 放在 flume 的 bin目录下，即 apache-flume-1.9.0-bin/bin/启动脚本**

```shell
#!/bin/bash
function start() {
    basepath=$(cd $(dirname $0);cd ..; pwd)
    echo $basepath
    f_cnt=`ps -ef |grep java|grep flume |grep -v grep| wc -l`
    if [ $f_cnt -le 0 ]; then
    # 进程不存在
        echo "start flume!"
    # 注意： 此处命令 需要根据具体配置做出修改, 尤其是  --name    --conf    --conf-file  等参数, 以及输出的 日志
        nohup $basepath/bin/flume-ng agent --name a1 --conf $basepath/conf/ --conf-file  $basepath/conf/flume-conf-test.properties > $basepath/logs/flume-start.log 2>&1 &
    else
    # 进程存在
        echo "flume already exists!"
    fi
}
 
function stop() {
    f_cnt=`ps -ef |grep java|grep flume |grep -v grep| wc -l`
    if [ $f_cnt -le 0 ]; then
    # 进程不存在
    echo "no flume running!"
    else
    # 进程存在
        ps -ef|grep java|grep flume|awk '{print $2;}'|xargs kill
    fi
}
 
 
case "$1" in
    start)
    echo "Starting flume Now......"
    start
    echo "Starting flume Finished"
    ;;
    stop)
    echo "Stopping flume Now......"
    stop
    echo "Stopping flume Finished"
    ;;
    restart)
    echo "Restart flume Now......"
    stop
    sleep 1
    start
    echo "Restart flume Finished"
    ;;
    *)
    echo $"usage: $0 {start | stop | restart}"
    exit 1
esac
```











