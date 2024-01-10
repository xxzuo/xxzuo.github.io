---
title: flume-User-Guide(1.11)(翻译中)
author: xxzuo
tags:
  - flume文档
categories:
  - flume
date: 2023-05-15 22:10:42
---
翻译自 [Flume 1.11.0 User Guide — Apache Flume](https://flume.apache.org/releases/content/1.11.0/FlumeUserGuide.html)

## Introduction介绍
### Overview概述
Apache Flume 是一个分布式、可靠和高可用的系统，用于有效地收集、聚合和移动来自许多不同来源的大量日志数据到一个集中的数据存储中。

使用 Apache Flume 不仅限于日志数据聚合。由于数据源是可定制的，因此 Flume 可用于传输大量事件数据，包括但不限于网络流量数据、社交媒体生成的数据、电子邮件消息和几乎任何可能的数据源。

Apache Flume 是 Apache Software Foundation 的顶级项目之一。

### System Requirements系统要求
1.  Java Runtime Environment - Java 1.8 or later[Java1.8+]
2.  Memory - Sufficient memory for configurations used by sources, channels or sinks[内存]
3.  Disk Space - Sufficient disk space for configurations used by channels or sinks[磁盘空间]
4.  Directory Permissions - Read/Write permissions for directories used by agent[目录权限]

### Architecture结构
#### Data flow model数据流模型
Flume Event 被定义为一个具有字节有效负载和可选的一组字符串属性的数据流单元。
Flume  Agent是一个 (JVM) 进程，托管事件从外部源流向下一个目标地点（跳跃）的组件。
![[Data-flow-model.excalidraw]]

##### Source
Flume source 消费 从外部源（如 web 服务器）传递给它的事件。
外部源以 目标 Flume Source 可识别的格式将事件发送到 Flume。
例如，可以使用 Avro Flume Source 接收来自 Avro 客户端或其他发送事件的 Flume 代理中的 Avro sink 的 Avro 事件。
可以使用 Thrift Flume Source 定义类似的流程，以从 Thrift Sink 或 Flume Thrift Rpc 客户端或使用 Flume thrift 协议生成的任何语言编写的 Thrift 客户端接收事件。
##### Channel
当 Flume Source 接收到事件时，它将其存储到一个或多个 Channel 中。Channel 是一个被动存储区，保留事件直到它被 Flume sink 消费。
比如 文件通道——它是由本地文件系统支持的。
##### Sink
Flume Sink 从通道中移除事件，并将其放入外部存储库，例如 HDFS（通过 Flume HDFS sink）或将其转发到流中下一个 Flume Agent 的 Flume source（下一跳）。在给定代理内部的源和 sink 异步运行，事件被暂存在通道中。
> 下一跳_(nexthop_)表示下一站数据包要到达的地址



#### Complex flows复杂流
Flume 允许用户构建 multi-hop(多跳) 流程，事件在到达最终目的地之前通过多个代理传输。它还允许扇入和扇出流程、上下文路由和备用路线（故障转移）以处理 failed hops (失败的跳点)。

#### Reliability可靠性
事件在每个 Agent 的 Channel 中进行。然后将事件传递到流中的下一个Agent 或 终端存储库（如HDFS）。只有在事件存储在下一个 Agent 的 Channel 或终端存储库中之后，才会从通道中删除这些事件。这就是Flume中的单跳消息传递语义如何提供流的 end-to-end reliability(端到端可靠性)。

Flume使用事务性方法来保证事件的可靠传递。Source 和 sink 分别封装了存储/检索的事务，并使用由 Channel 提供的事务存储或提供事件。这确保了一组事件可在流中从一个点可靠地传递到另一个点。在多跳流程的情况下，上一跳的 sink 和下一跳的 source 都在运行它们的事务，以确保数据安全地存储在下一跳的 Channel 中。

#### Recoverability可恢复性
事件被暂存在 Channel 中，并由 Channel 管理故障恢复。Flume 支持一个由本地文件系统支持的持久化文件通道。
还有一个 Memory Channel，它将事件存储在一个内存队列中，速度更快，但任何在 Agent 进程死亡时仍留在 Memory Channel(内存通道)中的事件都无法恢复。

Flume 的 KafkaChannel 使用 Apache Kafka 暂存事件。使用复制的 Kafka 主题作为通道可以帮助避免在磁盘故障的情况下丢失事件。



## Standard Setup标准设置
本节介绍 如何使用Flume长期以来的配置技术以及使用属性文件配置和连接Flume组件。
在下一节了解如何使用 Spring Boot 创建 Flume 应用程序。
### Setting up an agent设置Agent
Flume 代理配置存储在一个或多个遵循Java properties 文件格式的配置文件中。可以在这些配置文件中指定一个或多个代理的配置。配置包括 Agent 中每个 Source、Sink 和 Channel 的属性，以及它们如何连接在一起以形成数据流。
#### Configuring individual components配置单个组件
流中的每个组件（Source、Sink 或 Channel）都名称、类型和一组属性，这些属性特定于类型和实例化。
例如:
Avro Source 需要一个主机名（或IP地址）和一个端口号来接收数据。
Memory Channel 可以具有最大队列大小（"capacity"）
HDFS Sink 需要知道 文件系统URI、创建文件的路径、文件旋转频率（“HDFS.rollInterval”）等。
组件的所有这些属性都需要在托管Flume Agent 的 properties 文件中设置。

#### Wiring the pieces together连接组件
为了构成流，Agent 需要知道要加载哪些单独的组件以及它们是如何连接的。
这是通过列出代理中每个 Source、Sink 和 Channel 的名称来完成的，然后为每个 Source 和 Sink 指定 Channel。
例如:
Agent 将事件从名为 `avroWeb` 的 Avro Source 通过名为 `file-channel` 的 File Channel 流向名为 `hdfs-cluster1` 的 HDFS Sink。配置文件将包含这些组件的名称，并将 `file-channel` 作为 `avroWeb` Source 和 `hdfs-cluster1`  Sink 的共享 Channel。

#### Starting an agent启动Agent
Agent 程序是使用一个名为 `flume-ng` 的shell脚本启动的，该脚本位于 flume 发行版的 bin 目录中。您需要在命令行上指定代理名称、配置目录和配置文件：

```shell
bin/flume-ng agent -n $agent_name -c conf -f conf/flume-conf.properties.template
```
现在，Agent 将开始运行在给定属性文件(conf/flume-conf.properties.template) 中配置的 Source 和 Sink

#### A simple example
在这里，我们给出了一个示例配置文件，描述了单节点Flume部署。此配置允许用户生成事件，然后将其记录到控制台。

```properties
# example.conf: A single-node Flume configuration

# Name the components on this agent
a1.sources = r1
a1.sinks = k1
a1.channels = c1

# Describe/configure the source
a1.sources.r1.type = netcat
a1.sources.r1.bind = localhost
a1.sources.r1.port = 44444

# Describe the sink
a1.sinks.k1.type = logger

# Use a channel which buffers events in memory
a1.channels.c1.type = memory
a1.channels.c1.capacity = 1000
a1.channels.c1.transactionCapacity = 100

# Bind the source and sink to the channel
a1.sources.r1.channels = c1
a1.sinks.k1.channel = c1
```

此配置定义了一个名为a1的 Agent 。
a1有一个监听端口44444上数据的 Source.
一个在内存中缓冲事件数据的 Channel.
以及一个将事件数据记录到控制台的Sink
配置文件命名各种组件，然后描述它们的类型和配置参数。给定的配置文件可能定义了几个命名代理；当一个给定的Flume进程启动时，会传递一个标志，告诉它要显示哪个 Agent。

给定此配置文件，我们可以按如下方式启动Flume：

```shell
bin/flume-ng agent --conf conf --conf-file example.conf --name a1
```
请注意，在完整的部署中，我们通常会再包含一个选项：

> --conf=conf-dir

conf-dir目录将包括一个 shell 脚本flume-env.sh，并可能包括一个log4j配置文件(配置flume日志相关参数)。


从一个单独的终端，我们可以`telnet`端口44444并向Flume发送一个事件

```shell
$ telnet localhost 44444
Trying 127.0.0.1...
Connected to localhost.localdomain (127.0.0.1).
Escape character is '^]'.
Hello world! <ENTER>
OK
```
原始Flume终端将在日志消息中输出该事件。

```
12/06/19 15:32:19 INFO source.NetcatSource: Source starting
12/06/19 15:32:19 INFO source.NetcatSource: Created serverSocket:sun.nio.ch.ServerSocketChannelImpl[/127.0.0.1:44444]
12/06/19 15:32:34 INFO sink.LoggerSink: Event: { headers:{} body: 48 65 6C 6C 6F 20 77 6F 72 6C 64 21 0D          Hello world!. }
```
祝贺您-您已成功配置和部署Flume Agent！后面的章节将更详细地介绍 Agent 配置。

#### Configuration from URIs
从1.10.0版本开始，Flume支持使用URI进行配置，而不是仅从本地文件进行配置。包括对HTTP(S)、文件和类路径URI的直接支持。
HTTP支持包括对使用基本授权的身份验证的支持，但可以通过使用`–auth-provider`选项指定实现`AuthorizationProvider`接口的类的完全限定名称来支持其他授权机制。如果目标服务器正确响应`If-Modified-Since`标头，HTTP还支持使用轮询重新加载配置文件。


要指定HTTP身份验证的凭据，请添加：
> --conf-user userid --conf-password password

到启动命令。


#### Multiple Configuration Files
从1.10.0版本开始，Flume支持从多个配置文件进行配置，而不是仅从一个配置文件中进行配置。这样可以更容易地根据特定环境覆盖或添加值。每个文件都应该使用自己的–conf文件或–conf-uri选项进行配置。但是，所有文件都应该提供–conf文件或–conf-uri。如果–conf文件和–conf-uri一起显示为选项，则在合并任何–conf配置之前，将处理所有–conf-uri配置。


例如，以下各项的配置：

```shell
bin/flume-ng agent --conf conf --conf-file example.conf --conf-uri http://localhost:80/flume.conf --conf-uri http://localhost:80/override.conf --name a1
```

将导致首先读取 `flume.conf`，将 `override.conf` 与其合并，最后将 example.conf 合并到最后。
如果需要将 `example.conf` 作为基本配置,则应使用 `–conf-uri` 选项将其指定：

```shell
--conf-uri classpath://example.conf
or
--conf-uri file:///example.conf
```
这取决于应该如何访问它。

#### Using environment variables, system properies, or other properties configuration files使用环境变量、系统属性或其他属性配置文件
Flume能够替换配置中的环境变量。例如：
```properies
a1.sources = r1
a1.sources.r1.type = netcat
a1.sources.r1.bind = 0.0.0.0
a1.sources.r1.port = ${env:NC_PORT}
a1.sources.r1.channels = c1

```
注意：它目前只适用于值，不适用于键。（即仅在配置行的=标记的“右侧”。）

从1.10.0版本开始，Flume使用`Apache Commons Text`的`StringSubstitutior`类解析配置值，该类使用默认的查找集以及使用配置文件作为替换值源的查找。

例如：：
```shell
NC_PORT=44444 bin/flume-ng agent –conf conf –conf-file example.conf –name a1
```

注意，上面只是一个例子，环境变量可以通过其他方式配置，包括在conf/flome-env.sh中设置。

如前所述，还支持系统属性，因此配置：
```properies
a1.sources = r1
a1.sources.r1.type = netcat
a1.sources.r1.bind = 0.0.0.0
a1.sources.r1.port = ${sys:NC_PORT}
a1.sources.r1.channels = c1
```
可以使用，并且启动命令可以是：
```shell
bin/flume-ng agent --conf conf --conf-file example.conf --name a1 -DNC_PORT=44444
```

此外，由于允许多个配置文件，第一个文件可能包含：
```properies
a1.sources = r1
a1.sources.r1.type = netcat
a1.sources.r1.bind = 0.0.0.0
a1.sources.r1.port = ${NC_PORT}
a1.sources.r1.channels = c1
```
并且覆盖文件可以包含：
```properies
NC_PORT = 44444
```

在这种情况下，启动命令可以是：
```shell
bin/flume-ng agent --conf conf --conf-file example.conf --conf-file override.conf --name a1
```
请注意，如同先前版本中指定环境变量的方法仍然有效，但已被弃用，建议使用${env:varName}

#### Using a command options file使用命令选项文件
从1.10.0版本开始，命令选项可以放在类路径上的 `/etc/flume/flume.opts` 或`flume.opt`中，而不是在命令行上指定所有命令选项。例如：
```shell
conf-file = example.conf
conf-file = override.conf
name = a1
```


#### Logging raw data 记录原始数据
在许多生产环境中，记录流经摄取管道的原始数据流不是理想的行为，因为这可能会导致敏感数据或安全相关配置（如密钥）泄漏到 Flume 日志文件。默认情况下，Flume 不会记录此类信息。另一方面，如果数据管道中断，Flume 将尝试为调试问题提供线索。

调试事件管道问题的一种方法是设置一个连接到Logger接收器的额外内存通道，该通道将向Flume日志输出所有事件数据。然而，在某些情况下，这种方法是不够的。

为了能够记录与事件和配置相关的数据，除了log4j属性之外，还必须设置一些Java系统属性。

要启用与配置相关的日志记录，请设置Java系统属性
`-Dorg.apache.flume.log.printconfig=true`。这可以通过命令行传递，也可以在`flume-env.sh`中的`JAVA_OPTS`变量中设置。

要启用数据日志记录，请以与上述相同的方式设置Java系统属性
`-Dorg.apache.flume.log.rawdata=true`。对于大多数组件，log4j日志记录级别也必须设置为 DEBUG 或 TRACE，以使特定于事件的日志记录显示在Flume日志中。

以下是启用配置日志记录和原始数据日志记录的示例：
```shell
bin/flume-ng agent --conf conf --conf-file example.conf --name a1 -Dorg.apache.flume.log.printconfig=true -Dorg.apache.flume.log.rawdata=true
```

#### Zookeeper based Configuration zookeeper基础设置
Flum e通过 Zookeeper 支持 Agent 配置。这是一个实验特性。配置文件需要上传到 Zookeeper 中，并带有可配置的前缀。配置文件存储在Zookeeper节点数据中。以下是代理a1和a2的Zookeeper节点树的样子
```shell
- /flume
 |- /a1 [Agent config file]
 |- /a2 [Agent config file]
```
上传配置文件后，使用以下选项启动代理
```shell
bin/flume-ng agent –conf conf -z zkhost:2181,zkhost1:2181 -p /flume –name a1
```

| 参数名 | 默认值 | 描述                                                         |
| ------ | ------ | ------------------------------------------------------------ |
| z      | -      | Zookeeper connection string. 以逗号分隔的 hostname:port 列表 |
| p      | /flume | Zookeeper中存储代理配置的基本路径                            |


#### Installing third-party plugins安装第三方插件
Flume有一个完全基于插件的体系结构。虽然Flume提供了许多开箱即用的 Sources、Channels、Sinks、Serializers 等，但也存在许多与Flume分开提供的实现。

虽然一直可以通过将自定义Flume组件的jar添加到 `flume-env.sh` 文件中的 `FLUME_CLASSPATH` 变量来包含这些组件，但Flume现在支持一个名为 `plugins.d` 的特殊目录，该目录可以自动找到以特定格式打包的插件。这样可以更容易地管理插件打包问题，也可以更简单地调试和排除一些问题，尤其是 library 依赖关系冲突的问题。

##### The plugins.d directory plugins.d目录
`plugins.d`目录位于`$FLUME_HOME/plugins.d`。在启动时，`flume-ng`启动脚本在`plugins-d`目录中查找符合以下格式的插件，并在启动java时将其包含在正确的路径中。

##### Directory layout for plugins插件的目录布局
`plugins.d`中的每个插件（子目录）最多可以有三个子目录：
1.  lib - 插件的jar
2.  libext - 插件的依赖jar
3.  native - 任何必需的 native libraries, 比如.so文件

plugins.d目录中两个插件的示例:
```shell
plugins.d/
plugins.d/custom-source-1/
plugins.d/custom-source-1/lib/my-source.jar
plugins.d/custom-source-1/libext/spring-core-2.5.6.jar
plugins.d/custom-source-2/
plugins.d/custom-source-2/lib/custom.jar
plugins.d/custom-source-2/native/gettext.so
```


### Data ingestion数据摄入
Flume支持许多从外部来源获取数据的机制。
#### RPC
Flume 发行版中包含的 Avro客户端 可以使用Avro RPC 机制向Flume Avro Source 发送给定的文件：
```shell
bin/flume-ng avro-client -H localhost -p 41414 -F /usr/logs/log.10
```
上面的命令将 /usr/logs/log.10 到的内容发送到在该端口上监听的 Flume Source。

#### Executing commands执行命令
有一个`exec source`执行给定的命令并消耗输出。输出的单个“行”，即后面跟着回车（'\r'）或换行（'\n'）的文本，或两者同时出现。

#### Network streams网络流
Flume支持以下机制从流行的日志流类型中读取数据，例如：
1.  Avro
2.  Thrift
3.  Syslog
4.  Netcat

### Setting multi-agent flow设置多agent流
为了在多个 Agent 或 hops 之间传输数据，前一个 Agent 的 Sink 和当前 hops 的 Source 需要是avro类型，Sink 指向 Source 的主机名（或IP地址）和端口。
![[setting-multi-agent-flow.png]]

### Consolidation合并
日志收集中一个非常常见的场景是，大量生成日志的客户端将日志数据发送到连接到存储子系统的少数消费者代理。例如，从数百个web服务器收集的日志被发送到十几个写入HDFS集群的 Flume Agent。
![[consolidation1.png]]

这可以在Flume中通过配置具有avro Sink 的多个第一层代理来实现，所有 Agent 都指向单个 Agent 的avro Source（同样，在这种情况下，您可以使用 thrift sources/sinks/clients）。第二层 Agent 上的 Source 将接收到的 事件 合并到单个Channel 中，该通道由 Sink 消费到其最终目的地(如 HDFS)。


### Multiplexing the flow多路传输流
Flume支持将事件流(event flow)多路传输到一个或多个目的地。这是通过定义 流复用器(flow multiplexer) 来实现的，该流复用器可以将事件复制或选择性地路由到一个或多个channel。
![[multiplexing-the-flow1.png]]


上面的例子显示了一个来自 Agent “foo”的 Source，它将流分散到三个不同的Channel。这种扇出可以是复制或多路复用。在复制流的情况下，每个事件都会发送到所有三个Channel。对于多路复用情况，当事件的属性与预先配置的值匹配时，将事件传递到可用 Channel 的子集。
例如:
如果一个名为“txType”的事件属性设置为“customer”，那么它应该转到channel1和channel3，如果它是“vendor”，那么应该转到channel 2，否则转到channel3。可以在代理的配置文件中设置映射。


## Spring Boot Setup
ApacheFlume提供了`flume-spring-boot`模块，为使用`Spring Boot`打包和配置应用程序提供支持。应使用2.0.0版或更高版本`flume-spring-boot`。

### Creating the application创建应用
Flume的Spring Boot支持提供了要使用Spring配置的主类`org.apache.flume.spring.boot.FlumeApplication`。使用`Spring Boot`的Flume应用程序应该将其配置为`Spring Boot Maven plugin`的主类，如下所示：
```xml
<execution>
  <id>repackage</id>
  <goals>
    <goal>repackage</goal>
  </goals>
  <configuration>
    <executable>true</executable>
	<mainClass>org.apache.flume.spring.boot.FlumeApplication</mainClass>
  </configuration>
</execution>
```

### Component Scanning组件扫描
Spring Boot将自动定位Flume提供的所有Spring组件。然而为了配置Flume应用程序，Spring需要应用程序使用的配置和包名，以便Spring定位这些组件。这是在应用程序中通过提供`META-INF/spring.factories`文件来实现的，该文件允许对一个类进行自动配置，然后为应用程序的其余部分提供组件扫描信息(自动注入)。例如：
```xml
org.springframework.boot.autoconfigure.EnableAutoConfiguration=com.sample.myapp.config.AppConfig
```

`com.sample.config.AppConfig.java`:
```java
package com.sample.myapp.config;

import org.springframework.context.annotation.ComponentScan;
import org.springframework.context.annotation.Configuration;

@Configuration
@ComponentScan(basePackages="com.sample.myapp")
public class MyConfiguration {

}
```
这将导致`com.sample.myapp`包中的所有类及其子包被Spring扫描以查找要包含的组件。请注意，在那里找到的类也可能使用Spring的`@Import`注解将类包括在其他包中。

### Component Wiring组件连接
Flume的Spring Boot支持将自动收集所有定义的Channel、SourceRunners和SinkRunners并启动它们。要做到这一点，必须首先在类声明中包含`@Configuration`注解的类中使用Spring `@Bean`注解将它们创建为 `Spring Singletons`，然后像 “normal” `FlumeApplication`类那样初始化它们。要定义这些组件，应用程序应该提供一个创建这些Flume组件的`Configuration`类。生成序列号，将其写入Memory Channel，
使用这些事件而不在任何地方发布的示例配置如下：
```java
@Configuration
@ComponentScan(basePackages="com.sample.myapp")
public class AppConfig extends AbstractFlumeConfiguration {

  @Bean
  @ConfigurationProperties(prefix = "flume.sources.source1")
  public Map<String, String> source1Properties() {
    return new HashMap<>();
  }

  @Bean
  @ConfigurationProperties(prefix = "flume.channels.channel1")
  public Map<String, String> channel1Properties() {
    return new HashMap<>();
  }

  @Bean
  public Channel memoryChannel(Map<String, String> channel1Properties) {
    return configureChannel("channel1", MemoryChannel.class, channel1Properties);
  }

  @Bean
  public SourceRunner seqSource(Channel memoryChannel, Map<String, String> source1Properties) {
    ChannelSelector selector = new ReplicatingChannelSelector();
    selector.setChannels(listOf(memoryChannel));
    return configureSource("source1", SequenceGeneratorSource.class, selector,
        source1Properties);
  }

  @Bean
  public SinkRunner nullSink(Channel memoryChannel) {
    Sink sink = configureSink("null", NullSink.class, memoryChannel,null);
    return createSinkRunner(configureSinkProcessor(null, DefaultSinkProcessor.class,
        listOf(sink)));
  }
}
```

此配置可能如下所示:
```yaml
spring:
  application:
    name: flume-test

server:
  port: 41414

flume:
  metrics: http
  sources:
    source1:
      totalEvents: 10000
  channels:
    channel1:
      capacity: 10000
```

这将导致一个名为“flume test”的应用程序，该应用程序在端口41414上监听 /metrics 端点。10000个事件将被写入 Channel。这些事件将由 NullSink 使用。Configuration 类应该扩展AbstractFlumeConfiguration，如图所示，以便能够使用构建适当Flume组件的辅助类。
请注意，所有Spring Boot功能都可用于以这种方式配置的Flume应用程序。
SinkGroups 和 Sinks 也可以用与以下类似的方式进行配置：
```yaml
flume:
  sinkGroups:
    rrobin:
       backoff: true
       selector: round_robin
       "selector.maxTimeOut": 30000

  sinks:
    avroSinks:
      avroSink1:
        hostname: 192.168.10.10
        port: 4141
        batch-size: 100
        compression-type: deflate
      avroSink2:
        hostname: 192.168.10.11
        port: 4141
        batch-size: 100
        compression-type: deflate
```
这些将在Java配置中配置为：
```java
@Bean
@ConfigurationProperties(prefix = "flume.sink-groups.rrobin")
public Map<String, String> rrobinProperties() {
    return new HashMap<>();
}

@Bean
@ConfigurationProperties(prefix = "flume.sinks.avro-sinks")
public Map<String, AvroSinkConfiguration> avroSinksProperties() {
    return new HashMap<>();
}

@Bean
public List<Sink> avroSinks(final Channel avroFileChannel,
    final Map<String, AvroSinkConfiguration> avroSinksProperties) {
    List<Sink> sinks = new ArrayList<>();
    for (Map.Entry<String, AvroSinkConfiguration> entry : avroSinksProperties.entrySet()) {
        sinks.add(configureSink(entry.getKey(), AvroSink.class, avroFileChannel,
            entry.getValue().getProperties()));
    }
    return sinks;
}

@Bean
public SinkRunner avroSinkRunner(final Map<String, String> rrobinProperties, final List<Sink> avroSinks) {
    return createSinkRunner(
        configureSinkProcessor(rrobinProperties, LoadBalancingSinkProcessor.class, avroSinks));
}
```

请注意，为Source、Channel和 Sink组 指定的属性名称必须与本文档其他部分中为该组件指定的属性名相匹配。

需要注意的是，使用了一个具体的类来捕获Avro Sinks的数据。当使用简单的Map时，Spring似乎会与嵌套的Maps混淆。
AvroSinkConfiguration类：
```java
public class AvroSinkConfiguration {

   private String hostName;
   private int port;
   private int batchSize;
   private String compressionType;

   public String getHostName() {
      return hostName;
   }

   public void setHostName(String hostName) {
      this.hostName = hostName;
   }

   public int getPort() {
      return port;
   }

   public void setPort(int port) {
      this.port = port;
   }

   public int getBatchSize() {
      return batchSize;
   }

   public void setBatchSize(int batchSize) {
      this.batchSize = batchSize;
   }

   public String getCompressionType() {
      return compressionType;
   }

   public void setCompressionType(String compressionType) {
      this.compressionType = compressionType;
   }

   public Map<String, String> getProperties() {
      Map<String, String> map = new HashMap<>();
      map.put("hostname", hostName);
      map.put("port", Integer.toString(port));
      map.put("batchSize", Integer.toString(batchSize));
      map.put(compressionType, compressionType);
      return map;
   }
}
```


## Configuration
如前一节所述，标准Flume代理配置是从一个类似于具有分层属性设置的Java属性文件格式的文件中读取的。
### Defining the flow
要在单个 Agent 中定义流，需要通过 Channel 链接 Source 和 Sink。
需要列出给定 Agent 的Source、Sink和Channel，然后将 Source 和 Sink 指向 Channel。Source 实例可以指定多个 Sink，但 Sink 实例只能指定 `一个` Channel。格式如下：
```properties
# list the sources, sinks and channels for the agent
<Agent>.sources = <Source>
<Agent>.sinks = <Sink>
<Agent>.channels = <Channel1> <Channel2>

# set channel for source
<Agent>.sources.<Source>.channels = <Channel1> <Channel2> ...

# set channel for sink
<Agent>.sinks.<Sink>.channel = <Channel1>
```

例如，一个名为 agent_foo 的 Agent 正在从外部avro客户端读取数据，并通过 memory channel 将其发送到HDFS。配置文件 `weblog.config`可能如下所示：
```properties
# list the sources, sinks and channels for the agent
agent_foo.sources = avro-appserver-src-1
agent_foo.sinks = hdfs-sink-1
agent_foo.channels = mem-channel-1

# set channel for source
agent_foo.sources.avro-appserver-src-1.channels = mem-channel-1

# set channel for sink
agent_foo.sinks.hdfs-sink-1.channel = mem-channel-1
```

这将使事件通过 Memory Channel `mem-channel-1` 从 `avro-AppSrv-source` 流到`hdfs-Cluster1-sink`。当 Agent 以 `weblog.config` 作为其配置文件启动时，它将实例化该流。


### Configuring individual components 配置单个组件
定义流之后，需要设置每个Source、Sink和Channel 的属性。
这是以相同的分层命名空间方式完成的，您可以在其中为每个组件特定的属性设置组件类型和其他值：
```properties
# properties for sources
<Agent>.sources.<Source>.<someProperty> = <someValue>

# properties for channels
<Agent>.channels.<Channel>.<someProperty> = <someValue>

# properties for sinks
<Agent>.sinks.<Sink>.<someProperty> = <someValue>
```

Flume需要为每个组件设置属性“type”，以了解它需要什么样的对象。每个Source、Sink和Channel 类型都有自己的一组属性，这些属性是它按预期运行所需的。所有这些都需要根据需要进行设置。
在前面的示例中，我们有一个通过Memory Channel `mem-channel-1` 从 `avro-AppSrv-source` 流到`hdfs-Cluster1-sink`的流。下面是一个示例，显示了每个组件的配置：
```properties
agent_foo.sources = avro-AppSrv-source
agent_foo.sinks = hdfs-Cluster1-sink
agent_foo.channels = mem-channel-1

# set channel for sources, sinks

# properties of avro-AppSrv-source
agent_foo.sources.avro-AppSrv-source.type = avro
agent_foo.sources.avro-AppSrv-source.bind = localhost
agent_foo.sources.avro-AppSrv-source.port = 10000

# properties of mem-channel-1
agent_foo.channels.mem-channel-1.type = memory
agent_foo.channels.mem-channel-1.capacity = 1000
agent_foo.channels.mem-channel-1.transactionCapacity = 100

# properties of hdfs-Cluster1-sink
agent_foo.sinks.hdfs-Cluster1-sink.type = hdfs
agent_foo.sinks.hdfs-Cluster1-sink.hdfs.path = hdfs://namenode/flume/webdata

#...
```

### Adding multiple flows in an agent 在Agent中添加多个流
单个Flume Agent可以包含多个独立的流。您可以在一个配置中列出多个Source、Sink和Channel。这些组件可以链接以形成多个流：
```properties
# list the sources, sinks and channels for the agent
<Agent>.sources = <Source1> <Source2>
<Agent>.sinks = <Sink1> <Sink2>
<Agent>.channels = <Channel1> <Channel2>
```

然后，您可以将 Source 和 Sink 链接到它们对应的 Channel（对于 Source）或 Channel（对于Sink），以设置两个不同的流。例如，如果你需要在一个 Flume Agent 中设置两个流，一个从外部avro客户端到外部HDFS，另一个从文件尾部 输出到avro sink，那么这里有一个配置：
```properties
# list the sources, sinks and channels in the agent
agent_foo.sources = avro-AppSrv-source1 exec-tail-source2
agent_foo.sinks = hdfs-Cluster1-sink1 avro-forward-sink2
agent_foo.channels = mem-channel-1 file-channel-2

# flow #1 configuration
agent_foo.sources.avro-AppSrv-source1.channels = mem-channel-1
agent_foo.sinks.hdfs-Cluster1-sink1.channel = mem-channel-1

# flow #2 configuration
agent_foo.sources.exec-tail-source2.channels = file-channel-2
agent_foo.sinks.avro-forward-sink2.channel = file-channel-2
```

### Configuring a multi agent flow 配置多Agent流
要设置多层流，需要有一个第一跳的 avro/thrift sink 指向下一跳的 avro/thrift source 。这将导致第一个Flume Agent 将事件转发到下一个Flume Agent。例如，如果您使用avro客户端定期向本地Flume代理发送文件（每个事件1个文件），则此本地代理可以将其转发给另一个已装载用于存储的代理。
WebLog Agent 配置:
```properties
# list sources, sinks and channels in the agent
agent_foo.sources = avro-AppSrv-source
agent_foo.sinks = avro-forward-sink
agent_foo.channels = file-channel

# define the flow
agent_foo.sources.avro-AppSrv-source.channels = file-channel
agent_foo.sinks.avro-forward-sink.channel = file-channel

# avro sink properties
agent_foo.sinks.avro-forward-sink.type = avro
agent_foo.sinks.avro-forward-sink.hostname = 10.1.1.100
agent_foo.sinks.avro-forward-sink.port = 10000

# configure other pieces
#...
```

Hdfs Agent 配置:
```properties
# list sources, sinks and channels in the agent
agent_foo.sources = avro-collection-source
agent_foo.sinks = hdfs-sink
agent_foo.channels = mem-channel

# define the flow
agent_foo.sources.avro-collection-source.channels = mem-channel
agent_foo.sinks.hdfs-sink.channel = mem-channel

# avro source properties
agent_foo.sources.avro-collection-source.type = avro
agent_foo.sources.avro-collection-source.bind = 10.1.1.100
agent_foo.sources.avro-collection-source.port = 10000

# configure other pieces
#...
```

在这里，我们将来自weblog agent 的 `avro-forward-sink` 链接到hdfs agent的`avro-collection-source`。这将导致来自外部 appserver source 的 events 最终存储在HDFS中。


### Fan out flow
如前一节所述，Flume支持将流从一个 Source 扇出到多个 Channel。扇出有`复制`和`多路复用`两种模式。
在复制流中，事件被发送到所有配置的 Channel。
在多路复用的情况下，事件只发送到符合条件的 qualifying channels 的子集。
要扇出流，需要为 Source 指定一个 channels 列表，并指定扇出流的策略。
这是通过添加一个可以复制或多路复用的通道“selector”(选择器)来实现的。
如果是多路复用器，则进一步指定选择规则。
如果您没有指定选择器，那么默认情况下它正在复制:
```properties
# List the sources, sinks and channels for the agent
<Agent>.sources = <Source1>
<Agent>.sinks = <Sink1> <Sink2>
<Agent>.channels = <Channel1> <Channel2>

# set list of channels for source (separated by space)
<Agent>.sources.<Source1>.channels = <Channel1> <Channel2>

# set channel for sinks
<Agent>.sinks.<Sink1>.channel = <Channel1>
<Agent>.sinks.<Sink2>.channel = <Channel2>

<Agent>.sources.<Source1>.selector.type = replicating
```

多路复用选择具有另一组属性以将流分开。
这需要指定一个事件属性到 channel 集合的映射。
selector(选择器)检查event header 中的每个配置属性。
如果它与指定的值匹配，那么该事件将发送到映射到该值的所有 channel。
如果不匹配，则将事件发送到配置为默认的 channel 集合, 配置如下
```properties
# Mapping for multiplexing selector
<Agent>.sources.<Source1>.selector.type = multiplexing
<Agent>.sources.<Source1>.selector.header = <someHeader>
<Agent>.sources.<Source1>.selector.mapping.<Value1> = <Channel1>
<Agent>.sources.<Source1>.selector.mapping.<Value2> = <Channel1> <Channel2>
<Agent>.sources.<Source1>.selector.mapping.<Value3> = <Channel2>
#...

<Agent>.sources.<Source1>.selector.default = <Channel2>
```

映射允许每个值的 channel 重叠。

以下示例具有一个多路复用到两个路径的单个流。名为 `agent_foo` 的 Agent 有一个avro Source和 两个分别链接到两个Sink 的 Channel：
```properties
# list the sources, sinks and channels in the agent
agent_foo.sources = avro-AppSrv-source1
agent_foo.sinks = hdfs-Cluster1-sink1 avro-forward-sink2
agent_foo.channels = mem-channel-1 file-channel-2

# set channels for source
agent_foo.sources.avro-AppSrv-source1.channels = mem-channel-1 file-channel-2

# set channel for sinks
agent_foo.sinks.hdfs-Cluster1-sink1.channel = mem-channel-1
agent_foo.sinks.avro-forward-sink2.channel = file-channel-2

# channel selector configuration
agent_foo.sources.avro-AppSrv-source1.selector.type = multiplexing
agent_foo.sources.avro-AppSrv-source1.selector.header = State
agent_foo.sources.avro-AppSrv-source1.selector.mapping.CA = mem-channel-1
agent_foo.sources.avro-AppSrv-source1.selector.mapping.AZ = file-channel-2
agent_foo.sources.avro-AppSrv-source1.selector.mapping.NY = mem-channel-1 file-channel-2
agent_foo.sources.avro-AppSrv-source1.selector.default = mem-channel-1
```

上述配置 图示如下:
![[multiplexed-example-1.excalidraw]]

selector(选择器) 检查一个名为“State”的 header。
如果值为“CA”，则将其发送到 `mem-channel-1`
如果值为“AZ”，则发送到 `file-channel-2`
如果值为“NY”，那么两者都发送。
如果“State”标头未设置或与这三个标头中的任何一个都不匹配，则它将转到指定为“default”的`mem-channel-1`。

selector(选择器)还支持可选 channel。要为 header 指定可选通道，配置参数“optional”按以下方式使用：
```properties
# channel selector configuration
agent_foo.sources.avro-AppSrv-source1.selector.type = multiplexing
agent_foo.sources.avro-AppSrv-source1.selector.header = State
agent_foo.sources.avro-AppSrv-source1.selector.mapping.CA = mem-channel-1
agent_foo.sources.avro-AppSrv-source1.selector.mapping.AZ = file-channel-2
agent_foo.sources.avro-AppSrv-source1.selector.mapping.NY = mem-channel-1 file-channel-2
agent_foo.sources.avro-AppSrv-source1.selector.optional.CA = mem-channel-1 file-channel-2
agent_foo.sources.avro-AppSrv-source1.selector.mapping.AZ = file-channel-2
agent_foo.sources.avro-AppSrv-source1.selector.default = mem-channel-1
```

选择器将首先尝试写入 required channels
如果其中一个 Channel 无法使用事件，则会使事务失败。在所有 Channel 上重新尝试该事务。
一旦所有 required channels 都消耗了事件，选择器将尝试写入 optional channel。任何optional channel 使用事件的失败都将被忽略，并且不会重试。

如果特定 header 的 optional channels 和 required channels 之间存在重叠，则认为该 channel 是必需的，并且 channel 中的故障将导致 required channels 的整个集合被重试。例如，在上面的例子中，对于 header “CA”，mem-channel-1被认为是一个 required channels，即使它被标记为 required 和 optional，并且如果无法写入该 channel，则会导致在为选择器配置的所有 channel 上重试该事件。

请注意:
如果 header 没有任何 required channels，则事件将被写入默认 channels，并尝试写入该 header 的 optional channels。
如果未指定 required channels，即使指定  optional channels 仍会将事件写入默认 channel。
如果没有 channel 被指定为默认 channel，也没有required channels，选择器将尝试将事件写入 optional channels。在这种情况下，任何故障都会被忽略。


### SSL/TLS support

^cf1b41

几个Flume组件支持SSL/TLS协议，以便安全地与其他系统通信。

| 组件                        | SSL server or client |
| --------------------------- | -------------------- |
| Avro Source                 | server               |
| Avro Sink                   | client               |
| Thrift Source               | server               |
| Thrift Sink                 | client               |
| Kafka Source                | client               |
| Kafka Channel               | client               |
| Kafka Sink                  | client               |
| HTTP Source                 | server               |
| JMS Source                  | client               |
| Syslog TCP Source           | server               |
| Multiport Syslog TCP Source | server               |

SSL兼容组件有几个配置参数来设置SSL，如启用SSL标志、keystore / truststore 参数(location, password, type) 和其他SSL参数（例如 disabled protocols）。

在代理配置文件中，始终在组件级别指定启用 SSL。因此，有些组件可能被配置为使用 SSL，而其他组件则不使用（即使它们是相同的组件类型）。

keystore / truststore 设置可以在组件级别指定，也可以全局指定。
在组件级别设置的情况下，通过特定于组件的参数在 Agent 配置文件中配置 keystore / truststore。
这种方法的优点是组件可以使用不同的 keystore（如果需要的话）。缺点是必须为Agent 配置文件中的每个组件复制 keystore 参数。
组件级设置是可选的，但如果定义了它，它的优先级将高于全局参数。
通过全局设置，只需定义一次 keystore / truststore 参数，并对所有组件使用相同的设置就足够了，这意味着配置越来越集中。
全局设置可以通过系统属性或环境变量进行配置。

| System property系统属性          | Environment variable环境变量   | 描述                                                                                                       |
| -------------------------------- | ------------------------------ | ---------------------------------------------------------------------------------------------------------- |
| javax.net.ssl.keyStore           | FLUME_SSL_KEYSTORE_PATH        | Keystore location                                                                                          |
| javax.net.ssl.keyStorePassword   | FLUME_SSL_KEYSTORE_PASSWORD    | Keystore password                                                                                          |
| javax.net.ssl.keyStoreType       | FLUME_SSL_KEYSTORE_TYPE        | Keystore type (by default JKS)                                                                             |
| javax.net.ssl.trustStore         | FLUME_SSL_TRUSTSTORE_PATH      | Truststore location                                                                                        |
| javax.net.ssl.trustStorePassword | FLUME_SSL_TRUSTSTORE_PASSWORD  | Truststore password                                                                                        |
| javax.net.ssl.trustStoreType     | FLUME_SSL_TRUSTSTORE_TYPE      | Truststore type (by default JKS)                                                                           |
| flume.ssl.include.protocols      | FLUME_SSL_INCLUDE_PROTOCOLS    | 计算启用的协议时要包括的协议。以逗号（，）分隔的列表。排除的协议将从该列表中排除（如果提供）。             |
| flume.ssl.exclude.protocols      | FLUME_SSL_EXCLUDE_PROTOCOLS    | 计算启用的协议时要排除的协议。以逗号（，）分隔的列表。                                                     |
| flume.ssl.include.cipherSuites   | FLUME_SSL_INCLUDE_CIPHERSUITES | 计算启用的密码套件时要包括的密码套件。以逗号（，）分隔的列表。排除的密码套件将从该列表中排除（如果提供）。 |
| flume.ssl.exclude.cipherSuites   | FLUME_SSL_EXCLUDE_CIPHERSUITES | 计算启用的密码套件时要排除的密码套件。以逗号（，）分隔的列表。                                             |

SSL系统属性可以在命令行上传递，也可以通过在`conf/flume-env.sh`中设置 `JAVA_OPTS` 环境变量来传递(尽管如此，使用命令行是不可取的，因为包括密码在内的命令将保存到命令历史记录中。)
```shell
export JAVA_OPTS="$JAVA_OPTS -Djavax.net.ssl.keyStore=/path/to/keystore.jks"
export JAVA_OPTS="$JAVA_OPTS -Djavax.net.ssl.keyStorePassword=password"
```

Flume使用在JSSE(Java Secure Socket Extension)中定义的系统属性，因此这是设置SSL的标准方法。另一方面，在系统属性中指定密码意味着可以在进程列表中看到密码。对于不可接受的情况，也可以在环境变量中定义参数。在这种情况下，Flume在内部从相应的环境变量初始化JSSE系统属性。

SSL 环境变量可以在启动 Flume 之前在 shell 环境中设置，也可以在 `conf/flume-env.sh` 文件中设置。（尽管使用命令行是不可取的，因为包括密码在内的命令将保存到命令历史记录中。）
```shell
export FLUME_SSL_KEYSTORE_PATH=/path/to/keystore.jks
export FLUME_SSL_KEYSTORE_PASSWORD=password
```

**请注意:**
- 必须在组件级别启用SSL。单独指定全局SSL参数不会有任何效果。
- 如果在多个级别指定全局SSL参数，则优先级如下(从高到低):
	- component parameters in agent config代理配置中的组件参数
	- system properties系统属性
	- environment variables环境变量
- 如果为组件启用了SSL，但没有以上述任何方式指定SSL参数，则
	- 对于keystores密钥库：配置错误
	- 对于truststores信任库：将使用默认的信任库（Oracle JDK中的`jssecacerts / cacerts`）
- truststores信任库 密码在任何情况下都是可选的。如果未指定，那么当JDK打开 truststores信任库 时，将不会对其执行完整性检查。

### Source and sink batch sizes and channel transaction capacities Source和 Sink 批量大小以及 channel 处理容量
Source and sink 可以有一个批处理大小参数，该参数确定它们在一个批中处理的最大事件数。这种情况发生在具有称为事务容量的上限的 channel 事务中。批量大小必须小于 channel 的事务处理容量。有一个明确的检查来防止不兼容的设置。每当读取配置时，都会进行此检查。


### Flume Sources
![[Flume Sources]]



### Flume Sinks
![[Flume Sinks]]


### Flume Channels
