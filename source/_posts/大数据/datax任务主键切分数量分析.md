---
title: datax任务主键切分数量分析
author: xxzuo
tags:
  - datax
categories:
  - 大数据
date: 2023-04-13 21:05:43
---

在执行 某个离线同步任务时， 数据库告警，发现是datax查询 没有做分页
直接告警 
```sql
select * from table
```
目前对大表 不允许 直接 `select` 不加 `where` 条件
因此考虑 进行切片
datax 在执行任务时，支持对主键做切分

[DataX/mysqlreader/doc/mysqlreader.md](https://github.com/alibaba/DataX/blob/master/mysqlreader/doc/mysqlreader.md)
> - **splitPk**
    - 描述：MysqlReader进行数据抽取时，如果指定splitPk，表示用户希望使用splitPk代表的字段进行数据分片，DataX因此会启动并发任务进行数据同步，这样可以大大提供数据同步的效能。
        推荐splitPk用户使用表主键，因为表主键通常情况下比较均匀，因此切分出来的分片也不容易出现数据热点。
     目前splitPk仅支持整形数据切分，`不支持浮点、字符串、日期等其他类型`。如果用户指定其他非支持类型，MysqlReader将报错！
    如果splitPk不填写，包括不提供splitPk或者splitPk值为空，DataX视作使用单通道同步该表数据。
    - 必选：否  
    - 默认值：空
> 

通过代码 https://github.com/alibaba/DataX/blob/9626738ca8d60160bf6292abc8dd48015f3efc15/plugin-rdbms-util/src/main/java/com/alibaba/datax/plugin/rdbms/reader/util/SingleTableSplitUtil.java#L72
发现 
```java
    public static List<String> splitAndWrap(BigInteger left, BigInteger right, int expectSliceNumber, String columnName) {
        BigInteger[] tempResult = RangeSplitUtil.doBigIntegerSplit(left, right, expectSliceNumber);
        return RdbmsRangeSplitWrap.wrapRange(tempResult, columnName);
    }
```

切分是 根据 最大值、 最小值 还有 一个 `expectSliceNumber` 来的
而这个  `expectSliceNumber` 来自于 
https://github.com/alibaba/DataX/blob/9626738ca8d60160bf6292abc8dd48015f3efc15/plugin-rdbms-util/src/main/java/com/alibaba/datax/plugin/rdbms/reader/util/ReaderSplitUtil.java#L83

```java
if (tables.size() == 1) {  
    //原来:如果是单表的，主键切分num=num*2+1  
    // splitPk is null这类的情况的数据量本身就比真实数据量少很多, 和channel大小比率关系时，不建议考虑  
    //eachTableShouldSplittedNumber = eachTableShouldSplittedNumber * 2 + 1;// 不应该加1导致长尾  
    //考虑其他比率数字?(splitPk is null, 忽略此长尾)  
    //eachTableShouldSplittedNumber = eachTableShouldSplittedNumber * 5;  
    //为避免导入hive小文件 默认基数为5，可以通过 splitFactor 配置基数  
    // 最终task数为(channel/tableNum)向上取整*splitFactor  
    Integer splitFactor = originalSliceConfig.getInt(Key.SPLIT_FACTOR, Constant.SPLIT_FACTOR);  
    eachTableShouldSplittedNumber = eachTableShouldSplittedNumber * splitFactor;  
}
```

```java
        int eachTableShouldSplittedNumber = -1;
        if (isTableMode) {
            // adviceNumber这里是channel数量大小, 即datax并发task数量
            // eachTableShouldSplittedNumber是单表应该切分的份数, 向上取整可能和adviceNumber没有比例关系了已经
            eachTableShouldSplittedNumber = calculateEachTableShouldSplittedNumber(
                    adviceNumber, originalSliceConfig.getInt(Constant.TABLE_NUMBER_MARK));
        }
```


```java
    private static int calculateEachTableShouldSplittedNumber(int adviceNumber,
                                                              int tableNumber) {
        double tempNum = 1.0 * adviceNumber / tableNumber;

        return (int) Math.ceil(tempNum);
    }
```

而这个 一开始的 `adviceNumber` 来自于
https://github.com/alibaba/DataX/blob/9626738ca8d60160bf6292abc8dd48015f3efc15/core/src/main/java/com/alibaba/datax/core/job/JobContainer.java#L387
```java
this.adjustChannelNumber();  
if (this.needChannelNumber <= 0) {  
    this.needChannelNumber = 1;  
}  
List<Configuration> readerTaskConfigs = this  
        .doReaderSplit(this.needChannelNumber);
```

```java
private void adjustChannelNumber() {
        int needChannelNumberByByte = Integer.MAX_VALUE;
        int needChannelNumberByRecord = Integer.MAX_VALUE;

        boolean isByteLimit = (this.configuration.getInt(
                CoreConstant.DATAX_JOB_SETTING_SPEED_BYTE, 0) > 0);
        if (isByteLimit) {
            long globalLimitedByteSpeed = this.configuration.getInt(
                    CoreConstant.DATAX_JOB_SETTING_SPEED_BYTE, 10 * 1024 * 1024);

            // 在byte流控情况下，单个Channel流量最大值必须设置，否则报错！
            Long channelLimitedByteSpeed = this.configuration
                    .getLong(CoreConstant.DATAX_CORE_TRANSPORT_CHANNEL_SPEED_BYTE);
            if (channelLimitedByteSpeed == null || channelLimitedByteSpeed <= 0) {
                throw DataXException.asDataXException(
                        FrameworkErrorCode.CONFIG_ERROR,
                        "在有总bps限速条件下，单个channel的bps值不能为空，也不能为非正数");
            }

            needChannelNumberByByte =
                    (int) (globalLimitedByteSpeed / channelLimitedByteSpeed);
            needChannelNumberByByte =
                    needChannelNumberByByte > 0 ? needChannelNumberByByte : 1;
            LOG.info("Job set Max-Byte-Speed to " + globalLimitedByteSpeed + " bytes.");
        }

        boolean isRecordLimit = (this.configuration.getInt(
                CoreConstant.DATAX_JOB_SETTING_SPEED_RECORD, 0)) > 0;
        if (isRecordLimit) {
            long globalLimitedRecordSpeed = this.configuration.getInt(
                    CoreConstant.DATAX_JOB_SETTING_SPEED_RECORD, 100000);

            Long channelLimitedRecordSpeed = this.configuration.getLong(
                    CoreConstant.DATAX_CORE_TRANSPORT_CHANNEL_SPEED_RECORD);
            if (channelLimitedRecordSpeed == null || channelLimitedRecordSpeed <= 0) {
                throw DataXException.asDataXException(FrameworkErrorCode.CONFIG_ERROR,
                        "在有总tps限速条件下，单个channel的tps值不能为空，也不能为非正数");
            }

            needChannelNumberByRecord =
                    (int) (globalLimitedRecordSpeed / channelLimitedRecordSpeed);
            needChannelNumberByRecord =
                    needChannelNumberByRecord > 0 ? needChannelNumberByRecord : 1;
            LOG.info("Job set Max-Record-Speed to " + globalLimitedRecordSpeed + " records.");
        }

        // 取较小值
        this.needChannelNumber = needChannelNumberByByte < needChannelNumberByRecord ?
                needChannelNumberByByte : needChannelNumberByRecord;

        // 如果从byte或record上设置了needChannelNumber则退出
        if (this.needChannelNumber < Integer.MAX_VALUE) {
            return;
        }

        boolean isChannelLimit = (this.configuration.getInt(
                CoreConstant.DATAX_JOB_SETTING_SPEED_CHANNEL, 0) > 0);
        if (isChannelLimit) {
            this.needChannelNumber = this.configuration.getInt(
                    CoreConstant.DATAX_JOB_SETTING_SPEED_CHANNEL);

            LOG.info("Job set Channel-Number to " + this.needChannelNumber
                    + " channels.");

            return;
        }

        throw DataXException.asDataXException(
                FrameworkErrorCode.CONFIG_ERROR,
                "Job运行速度必须设置");
    }

```


从上述代码可以发现 ， 单表场景下 最终 任务切分的数量是 
`channel * splitFactor + 1` 
> 注意: 多出的 1个任务 是 `SELECT * FROM TABLE WHERE ID IS NULL` 这个SQL

