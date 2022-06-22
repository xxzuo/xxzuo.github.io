---
title: 创建hive-udf
author: xxzuo
date: 2022-06-22 23:28:20
tags:
- hive
categories:
- 大数据
---



### 1.自定义 UDF 函数
pom 中添加下面依赖
```xml
<dependencies>
    <dependency>
        <groupId>org.apache.hive</groupId>
        <artifactId>hive-exec</artifactId>
        <version>2.3.4</version>
        <!-- 不需要打入jar包 -->
        <scope>provided</scope>
    </dependency>
</dependencies>
```
> 由于  hive  环境中已经有了  hive-exec jar包 所以打包时，不需要将hive-exec 打入

新建一个类 继承 UDF 即可
```java
// 继承UDF类
public class MyUDF extends UDF {
// 实现evaluate方法
    public int evaluate(String str) {
        // 自定义逻辑
        return 0;
    }
}
```

在这个类中，方法名一定要为 **evaluate**

### 2.打包上传
在 maven 中打包
找到刚才的 jar 包
上传到 hive 所在服务器

将jar包移动到  hive目录下的auxlib目录

然后重启hiveserver2


### 3.注册函数
将jar包注册到hive中后，就可以注册刚才自己编写的udf函数了

一般需要先注册临时函数，因为UDF开发完成后，需要进过一些测试才能确认代码是否没有问题。在测试UDF代码时，务必使用临时函数进行测试。这样即使代码出现了问题，也不会把函数真正的注册到Hive。代码测试完毕并且确认无误后，再将函数注册为永久函数


HIVE 函数相关
```sql
# 创建临时函数
CREATE TEMPORARY FUNCTION <函数名> AS <class全路径>;
# 删除临时函数
DROP TEMPORARY FUNCTION [IF EXISTS] <函数名>;


# 创建永久函数
CREATE FUNCTION [db_name.]function_name AS class_name
  [USING JAR|FILE|ARCHIVE 'file_uri' [, JAR|FILE|ARCHIVE 'file_uri'] ];
# 删除永久函数
DROP FUNCTION [IF EXISTS] function_name;


# 查看函数
show  functions;

```
