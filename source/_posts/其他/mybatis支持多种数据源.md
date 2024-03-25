---
title: mybatis支持多种数据源
author: xxzuo
tags:
  - java
categories:
  - 其他
date: 2024-02-25 19:28:32
---
有时候 代码中需要支持 多种数据源部署 (注意：非多数据源)
比如 同时支持 `pgsql` 和 `mysql`,  但因为某些 SQL 语法的不同，需要 根据不同的 数据库 执行不同的 SQL
这里 用到了  `databaseIdProvider`
[mybatis – MyBatis 3 | Configuration](https://mybatis.org/mybatis-3/configuration.html#databaseidprovider)

代码参考这个

```java
import com.baomidou.mybatisplus.annotation.DbType;  
import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;  
import com.baomidou.mybatisplus.extension.plugins.inner.PaginationInnerInterceptor;  
import org.apache.ibatis.mapping.DatabaseIdProvider;  
import org.apache.ibatis.mapping.VendorDatabaseIdProvider;  
import org.springframework.context.annotation.Bean;  
import org.springframework.context.annotation.Configuration;  
  
import java.util.Properties;  
  
@Configuration  
public class MybatisPlusConfig {  
  
    @Bean  
    public MybatisPlusInterceptor mybatisPlusInterceptor() {  
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();  
        interceptor.addInnerInterceptor(new PaginationInnerInterceptor(DbType.MYSQL));  
        return interceptor;  
    }  
  
    @Bean  
    public DatabaseIdProvider getDatabaseIdProvider(){  
        DatabaseIdProvider databaseIdProvider = new VendorDatabaseIdProvider();  
        Properties properties = new Properties();  
        properties.setProperty("Oracle","oracle");  
        properties.setProperty("MySQL","mysql");  
        properties.setProperty("DB2","db2");  
        properties.setProperty("Derby","derby");  
        properties.setProperty("H2","h2");  
        properties.setProperty("HSQL","hsql");  
        properties.setProperty("Informix","informix");  
        properties.setProperty("MS-SQL","ms-sql");  
        properties.setProperty("PostgreSQL","postgresql");  
        properties.setProperty("Sybase","sybase");  
        properties.setProperty("Hana","hana");  
        databaseIdProvider.setProperties(properties);  
        return databaseIdProvider;  
    }  
}
```


使用时 
```xml
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >  
<mapper namespace="io.datavines.server.repository.mapper.ConfigMapper">  
    <select id="getNow" resultType="java.lang.String"> 
	    select DATE_FORMAT(now(), '%Y-%m-%d') AS now_date
    </select>
        <select id="getNow" resultType="java.lang.String" databaseId="postgresql"> 
	    SELECT CURRENT_DATE;
    </select>  
</mapper>

```


这里  `properties` 设置的 `key` 和 `value` 中 
- `key` 是 从 `DatabaseMetaData.getDatabaseProductName()` 来， 不同数据库不一样
- `value` 是我们自定义的别名，通过别名我们可以在SQL语句中标识适用于哪种数据库运行