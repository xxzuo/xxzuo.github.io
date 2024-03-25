---
title: SQL生成连续列表
author: xxzuo
tags:
  - doris
  - sql
  - hive
categories:
  - SQL相关
date: 2024-03-21 20:50:33
---
有时候 需要根据 给定的  开始数据 和 结束数据，生成一段连续序列
比如 给定 开始日期 和 结束日期 生成 连续日期列表。
又或者 给定 开始数字 和 结束数字 生成连续数字列表
就像这样
```
start_date: 2024-01-01
end_date: 2024-01-10

res:
2024-01-01
2024-01-02
2024-01-03
2024-01-04
2024-01-05
2024-01-06
2024-01-07
2024-01-08
2024-01-09
2024-01-10
```

这里 一般只需要处理三个地方
- 计算 开始值 和 结束值 的 差值
- 根据 差值 生成连续序列
- 把 开始值 加上 连续序列的值

这里 以 doris 和 hive 来举例
### Doris生成连续日期

生成 `2024-01-01` 和 `2024-01-10`的连续日期序列

- 使用 `datediff`计算差值
[DATEDIFF - Apache Doris](https://doris.apache.org/zh-CN/docs/sql-manual/sql-functions/date-time-functions/datediff/)
```
select datediff('2024-01-10', '2024-01-01')
```

- 利用 explode_number 生成连续序列
[EXPLODE_NUMBERS - Apache Doris](https://doris.apache.org/zh-CN/docs/sql-manual/sql-functions/table-functions/explode-numbers/)
```
select e1
from (select '2024-01-01' k1) as t lateral view explode_numbers(datediff('2024-01-10', '2024-01-01')) tmp1 as e1;
```

- 开始值 加上 连续序列
[DAYS_ADD - Apache Doris](https://doris.apache.org/zh-CN/docs/sql-manual/sql-functions/date-time-functions/days-add/)
```
select e1, k1, to_date(days_add(k1, e1))  
from (select '2024-01-01' k1) as t lateral view explode_numbers(datediff('2024-01-10', '2024-01-01') + 1) tmp1 as e1;
```


