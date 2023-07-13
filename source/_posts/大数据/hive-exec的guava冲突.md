---
title: hive-exec的guava冲突
author: xxzuo
tags:
  - hive
categories:
  - 大数据
date: 2023-07-13 23:28:27
---

在 引入 `hive-exec` 的maven 依赖后 Application 无法启动
报 `hive-exec` 的 guava 与系统自带的 guava 冲突
此时 尝试在 maven 中 
```xml
<exclusion>  
   <groupId>com.google.guava</groupId>  
   <artifactId>guava</artifactId>  
</exclusion>
```
仍然无法启动

这是因为 `hive-exec `的 guava 是 父模块引入的
所以我们需要 去掉 guava 后重新编译 hive 

找到 `hive-exec` 模块的 `pom.xml` 文件
注释掉这个地方
```
<include>com.google.guava:guava</include>
```

然后 重新编译
编译 见 ![[hive2.1.1编译]]

编译 完以后 用自己编译出来的 `hive-exec` 的jar 包替换掉从 maven中央仓库中下载 `hive-exec` jar包
即可





