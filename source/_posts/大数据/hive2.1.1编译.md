---
title: hive编译
author: xxzuo
tags:
  - hive
categories:
  - 大数据
date: 2023-07-13 23:17:02
---

### 1.下载源码
由于 git 上 ([branch-2.1](https://github.com/apache/hive/tree/branch-2.1))的pom version 为 2.1.2 所以 需要从  [Release rel/release-2.1.1 · apache/hive (github.com)](https://github.com/apache/hive/releases/tag/rel%2Frelease-2.1.1)
下载源码

### 2.编译
切换到 hive 根目录下   编译
```cmd
 mvn clean package -DskipTests -Pdist
```
在 windows 下编译时,  hive-common 模块会报
```
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-antrun-plugin:1.7:run (generate-version-annotation) on project hive-common: An Ant BuildException has occured: exec returned: 127
[ERROR] around Ant part ...<exec failonerror="true" executable="bash">... @ 4:46 in D:\code_repo\java\hive-rel-release-2.1.1\hive-rel-release-2.1.1\common\target\antrun\build-main.xml
[ERROR] -> [Help 1]
[ERROR]
[ERROR] To see the full stack trace of the errors, re-run Maven with the -e switch.
[ERROR] Re-run Maven using the -X switch to enable full debug logging.
[ERROR]
[ERROR] For more information about the errors and possible solutions, please read the following articles:
[ERROR] [Help 1] http://cwiki.apache.org/confluence/display/MAVEN/MojoExecutionException
[ERROR]
[ERROR] After correcting the problems, you can resume the build with the command
[ERROR]   mvn <args> -rf :hive-common

```

这是因为 需要 bash 环境。这里我选择在 wsl2 下编译 (此处我 wsl2 的 maven 和 jdk 都是使用 wsl2上的,  而不是使用 windows 的 maven 和 jdk) (如果 wsl2 的 maven 指定的本地仓库 在 windows 上 此时 会非常慢 可能是因为 跨系统访问文件导致的)

再次执行
```bash
mvn clean package -DskipTests -Pdist
```
编译 成功











