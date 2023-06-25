---
title: yarn配置
author: xxzuo
tags:
  - yarn
categories:
  - 大数据
date: 2022-06-21 22:54:37
---



### Capacity Scheduler（容量调度器）
 ```xml
<property>
    <name>yarn.scheduler.capacity.maximum-am-resource-percent</name>
    <value>0.8</value>
</property>
 ```
>  maximum-am-resource-percent：集群中用于运行应用程序ApplicationMaster的资源比例上限，该参数通常用于限制处于活动状态的应用程序数目。所有队列的ApplicationMaster资源比例上限可通过参数yarn.scheduler.capacity.maximum-am-resource-percent设置，而单个队列可通过参数yarn.scheduler.capacity.<queue-path>.maximum-am-resource-percent设置适合自己的值


### 节点级别 

```xml
<property>
    <name>yarn.nodemanager.resource.memory-mb</name>
    <value>-1</value>
    <final>false</final>
    <source>yarn-default.xml</source>
</property>
```
> 说明：每个节点可用的最大内存，RM中的两个值不应该超过此值。此数值可以用于计算container最大数目，即：用此值除以RM中的最小容器内存。


```xml
<property>
    <name>yarn.nodemanager.resource.cpu-vcores</name>
    <value>-1</value>
    <final>false</final>
    <source>yarn-default.xml</source>
</property>
```
每个节点总的可用虚拟CPU个数。 默认值：8
> 目前推荐将该值设值为与物理CPU核数数目相同。如果你的节点CPU核数不够8个，则需要调减小这个值，而YARN不会智能的探测节点的物理CPU总数


```xml
<property>
    <name>yarn.nodemanager.vmem-pmem-ratio</name>
    <value>2.1</value>
    <final>false</final>
    <source>yarn-default.xml</source>
</property>
```
> Ratio between virtual memory to physical memory when setting memory limits for containers. Container allocations are expressed in terms of physical memory, and virtual memory usage is allowed to exceed this allocation by this ratio.

> 每单位的物理内存总量对应的虚拟内存量，默认是2.1，表示每使用1MB的物理内存，最多可以使用2.1MB的虚拟内存总量。



### 容器级别
```xml
<property>
    <name>yarn.scheduler.maximum-allocation-mb</name>
    <value>8192</value>
    <final>false</final>
    <source>yarn-default.xml</source>
</property>


<property>
    <name>yarn.scheduler.minimum-allocation-mb</name>
    <value>1024</value>
    <final>false</final>
    <source>yarn-default.xml</source>
</property>
```

> 说明：单个容器可申请的最小与最大内存，应用在运行申请内存时不能超过最大值，小于最小值则分配最小值


```xml
<property>
    <name>yarn.scheduler.minimum-allocation-vcores</name>
    <value>1</value>
    <final>false</final>
    <source>yarn-default.xml</source>
</property>


<property>
    <name>yarn.scheduler.maximum-allocation-vcores</name>
    <value>4</value>
    <final>false</final>
    <source>yarn-default.xml</source>
</property>
```

> 说明：单个容器可申请的最小/最大虚拟CPU个数。比如设置为1和4，则运行MapRedce作业时，每个Task最少可申请1个虚拟CPU，最多可申请4个虚拟CPU。


AM内存配置相关参数，此处以MapReduce为例进行说明（这两个值是AM特性，应在mapred-site.xml中配置），如下：

> mapreduce.map.memory.mb
> mapreduce.reduce.memory.mb

说明：这两个参数指定用于MapReduce的两个任务（Map and Reduce task）的内存大小，其值应该在RM中的最大最小container之间。如果没有配置则通过如下简单公式获得：

> max(MIN_CONTAINER_SIZE, (Total Available RAM) / containers))

一般的reduce应该是map的2倍。注：这两个值可以在应用启动时通过参数改变；

AM中其它与内存相关的参数，还有JVM相关的参数，这些参数可以通过，如下选项配置：

> mapreduce.map.java.opts
> mapreduce.reduce.java.opts

说明：这两个参主要是为需要运行JVM程序（java、scala等）准备的，通过这两个设置可以向JVM中传递参数的，与内存有关的是，-Xmx，-Xms等选项。此数值大小，应该在AM中的map.mb和reduce.mb之间。
