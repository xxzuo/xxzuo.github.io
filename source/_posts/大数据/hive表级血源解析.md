---
title: hive表级血源解析
author: xxzuo
tags:
  - hive
categories:
  - 大数据
date: 2023-07-13 23:29:13
---

### hive血源

通过查看代码 发现 hive 自带了 一个表级血源解析工具类
```
org.apache.hadoop.hive.ql.tools.LineageInfo
```

我们可以这样使用
```java
public static void main(String[] args) throws IOException, ParseException,  
        SemanticException {  
    String query = "INSERT OVERWRITE TABLE TABL_A SELECT * FROM TABLE_B";  
    LineageInfo lep = new LineageInfo();  
    lep.getLineageInfo(query);
    for (String tab : lep.getInputTableList()) {  
        System.out.println("InputTable=" + tab);  
    }  
    for (String tab : lep.getOutputTableList()) {  
        System.out.println("OutputTable=" + tab);  
    }  
}


```

这样 就可以根据 hive sql 获取 表级血源

使用时 需要引入依赖
```xml
<dependency>  
    <groupId>org.apache.hive</groupId>  
    <artifactId>hive-exec</artifactId>  
    <version>2.1.1</version>  
</dependency>
```





