---
title: flink读kafka数据写入hive
author: xxzuo
tags:
  - flink
  - kafka
  - hive
categories:
  - 大数据
date: 2022-08-04 00:17:58
---



### maven依赖
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.xxzuo</groupId>
    <artifactId>xxx</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>xxx</name>
    <description>xxx</description>

    <properties>
        <maven.compiler.source>1.8</maven.compiler.source>
        <maven.compiler.target>1.8</maven.compiler.target>
        <encoding>UTF-8</encoding>
        <scala.binary.version>2.11</scala.binary.version>
        <hadoop.version>3.2.2</hadoop.version>
        <hive.version>3.1.2</hive.version>
        <flink.version>1.14.4</flink.version>
        <druid.version>1.1.20</druid.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.24</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-clients_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>com.ververica</groupId>
            <artifactId>flink-connector-mysql-cdc</artifactId>
            <version>2.0.2</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-csv</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-base</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-jdbc_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-table-planner_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-kafka_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-json</artifactId>
            <version>${flink.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-table-api-java-bridge_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-sql-connector-kafka_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>


        <dependency>
            <groupId>org.apache.flink</groupId>
            <artifactId>flink-connector-hive_${scala.binary.version}</artifactId>
            <version>${flink.version}</version>
        </dependency>

        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-math3</artifactId>
            <version>3.6.1</version>
        </dependency>

        <dependency>
            <groupId>org.apache.hive</groupId>
            <artifactId>hive-exec</artifactId>
            <version>3.1.0</version>
        </dependency>

        <dependency>
            <groupId>org.apache.thrift</groupId>
            <artifactId>libfb303</artifactId>
            <version>0.9.3</version>
        </dependency>

        <dependency>
            <groupId>org.antlr</groupId>
            <artifactId>antlr-runtime</artifactId>
            <version>3.5.2</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.58</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>${druid.version}</version>
        </dependency>
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
            <version>3.9</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.13</version>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <!-- 编译插件 -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.6.0</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            <!-- 打jar包插件(会包含所有依赖) -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-assembly-plugin</artifactId>
                <version>2.6</version>
                <configuration>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                    <archive>
                        <manifest>
                            <!-- 可以设置jar包的入口类(可选) -->
                            <mainClass></mainClass>
                        </manifest>
                    </archive>
                </configuration>
                <executions>
                    <execution>
                        <id>make-assembly</id>
                        <phase>package</phase>
                        <goals>
                            <goal>single</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>

```

### kafka source
```java
String bootstrapServers = "192.168.1.1:9092,192.168.1.2:9092";
KafkaSource<String> source = KafkaSource.<String>builder()
        .setBootstrapServers(bootstrapServers)
        .setTopics("kafka_topic")
        .setGroupId("consumer_group")
        .setStartingOffsets(OffsetsInitializer.committedOffsets(OffsetResetStrategy.EARLIEST))
        .setValueOnlyDeserializer(new SimpleStringSchema())
        .build();
DataStreamSource<String> stream = env.fromSource(source, WatermarkStrategy.forMonotonousTimestamps(), "kafka_source");
```

### hive sink

#### 设置hiveCatalog
hiveCatalog作用是：使用 hive 的 metastore去管理 flink元数据 ，持久化元数据，避免每次使用时都要重新注册
代码中使用 hive catalog
```java
// 设置 catalog
String catalogName = "flinkHive";
// 路径为 hive 配置文件路径
HiveCatalog catalog = new HiveCatalog(
        catalogName,
        "flink",
        "/usr/local/hive/apache-hive-3.1.2-bin/conf/"
);
tableEnv.registerCatalog(catalogName, catalog);
// 使用注册的catalog
tableEnv.useCatalog(catalogName);
```
#### 将 kafka stream 映射为临时表
```java
// 建立临时视图 映射 kafka source
tableEnv.createTemporaryView("flink.kafka_table", stream);
```

#### 创建 hive 表
```java
// hive 建表时 需要切换 方言为 HIVE
tableEnv.getConfig().setSqlDialect(SqlDialect.HIVE);
tableEnv.executeSql(" CREATE TABLE if not exists flink.hive_table (\n" +
                        "hive_col1 STRING,\n" +
                        "hive_col2 STRING,\n" +
                        "           ) " +
                        "PARTITIONED BY (\n" +
                        "                     day STRING\n" +
                        "           ) " +
                        "STORED AS PARQUET\n" +
                        "             TBLPROPERTIES (\n" +
                        // partition-commit.trigger 触发分区提交
                        // process-time 根据处理时间触发
                        // partition-time 根据从事件时间中提取的分区时间触发
                        "                    'sink.partition-commit.trigger' = 'process-time',\n" +
                        // partition-commit.delay 提交的延迟时间
                        "                    'sink.partition-commit.delay' = '0s',\n" +
                        // partition-commit.policy.kind 分区提交策略
                        // metastore 提交到元数据
                        // success-file 写入_success文件到分区目录中
                        "                    'sink.partition-commit.policy.kind' = 'metastore,success-file',\n" +
                        // time-extractor.timestamp-pattern 指定分区提取器提取时间戳的格式 
                        "                    'partition.time-extractor.timestamp-pattern'='$day 00:00:00'" +
                        "           )"
        );
        // hive 建表完 把方言切换回 DEFAULT
        tableEnv.getConfig().setSqlDialect(SqlDialect.DEFAULT);
```

#### sink数据到 hive
```java
TableResult tr = tableEnv.executeSql(
                "INSERT INTO flink.hive_table " +
                        "SELECT " +
                        "col1,\n" +
                        "col2,\n" +
                        "datetime\n" +
                        " from flink.kafka_table A ");
```
