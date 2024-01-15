---
title: Flume-Sources文档(翻译中)
author: xxzuo
tags:
  - flume文档
categories:
  - flume
date: 2023-05-18 22:00:12
---
#### Avro Source
在Avro端口上监听并接收来自外部Avro客户端流的事件。当与另一个（上一跳）Flume Agent上的内置Avro Sink配对时，它可以创建分层收集拓扑。必填属性以粗体显示。

| Property Name         | Default | Description                                                                                                                                           |
| --------------------- | ------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| **channels**          | -       |                                                                                                                                                       |
| **type**              | -       | 组件类型名称，需要为avro                                                                                                                              |
| **bind**              | -       | 要监听的主机名或IP地址                                                                                                                                |
| **port**              | -       | 要绑定到的端口号                                                                                                                                      |
| threads               | -       | 生成的 worker 的最大线程数量                                                                                                                          |
| selector.type         |         |                                                                                                                                                       |
| selector.*            |         |                                                                                                                                                       |
| interceptors          | -       | 以空格分隔的拦截器列表                                                                                                                                |
| interceptors.*        |         |                                                                                                                                                       |
| compression-type      | none    | 这可以是“none” or “deflate”。压缩类型必须与匹配Avro Source的压缩类型匹配                                                                              |
| ssl                   | false   | 将此设置为true可启用SSL加密。如果启用了SSL，则还必须通过组件级参数（见下文）或全局SSL参数（见SSL/TLS支持部分）指定“keystore” 和 “keystore-password”。 |
| keystore              | -       | 这是指向 Java 密钥库文件的路径。如果没有在这里指定，则将使用全局密钥库（如果全局未定义 则会发生配置错误）。                                           |
| keystore-password     | -       | Java密钥库的密码。如果未在此处指定，则将使用全局密钥库密码（如果全局未定义 则会发生配置错误）。                                                       |
| keystore-type         | JKS     | Java密钥库的类型。这可以是“JKS”或“PKCS12”。如果此处未指定，则将使用全局密钥存储类型（如果全局未定义 则默认为JKS）。                                   |
| exclude-protocols     | SSLv3   | 要排除的SSL/TLS协议的空格分隔列表。除了指定的协议外，SSLv3将始终被排除在外。                                                                          |
| include-protocols     | -       | 要包含的SSL/TLS协议的空格分隔列表。启用的协议将是包含的协议，而没有排除的协议。如果包含的协议为空，则它包括所有支持的协议。                           |
| exclude-cipher-suites | -       | 要排除的以空格分隔的密码套件列表。                                                                                                                    |
| include-cipher-suites | -       | 要包含的以空格分隔的密码套件列表。启用的密码套件将是包含的密码套件，而不包含排除的密码套件。如果包含的密码套件为空，则它包括所有支持的密码套件。      |
| ipFilter              | false   | 将此设置为true以启用netty的ipFiltering                                                                                                                |
| ipFilterRules         | -       | 使用此配置定义N个netty ipFilter模式规则。                                                                                                             |

名为a1的 Agent 示例：
```properties
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = avro
a1.sources.r1.channels = c1
a1.sources.r1.bind = 0.0.0.0
a1.sources.r1.port = 4141
```

ipFilterRules 示例:
ipFilterRules 定义了 N 个 Netty IP 过滤器，用逗号分隔。一个模式规则必须采用下面的格式。
```
<’allow’ or deny>:<’ip’ or ‘name’ for computer name>:<pattern> or allow/deny:ip/name:patter
```
比如:
```properties
ipFilterRules=allow:ip:127.*,allow:name:localhost,deny:ip:*
```

请注意，第一个要匹配的规则将从本地主机上的客户端应用，如下例所示
这将允许本地主机上的客户端,拒绝来自任何其他ip的客户端
> allow:name:localhost,deny:ip:


这将拒绝本地主机上的客户端,允许来自任何其他ip的客户端
> deny:name:localhost,allow:ip:



#### Thrift Source
监听 Thrift 端口并接收来自外部 Thrift 客户端流的事件。当与另一个（上一跳）Flume Agent上的内置 ThriftSink 配对时，它可以创建分层集合拓扑。通过启用kerberos身份验证，可以将Thrift Source 配置为以安全模式启动。agent-principal 和 agent-keytab 是Thrift Source 用来向kerberos KDC进行身份验证的属性。必填属性以粗体显示。

| Property Name         | Default | Description                                                                                                                                                                                                      |
| --------------------- | ------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **channels**          | -       |                                                                                                                                                                                                                  |
| **type**              | -       | 组件类型名称，需要为thrift                                                                                                                                                                                       |
| **bind**              | -       | 要监听的主机名或IP地址                                                                                                                                                                                           |
| **port**              | -       | 要绑定到的端口号                                                                                                                                                                                                 |
| threads               | -       | 生成的 worker 的最大线程数量                                                                                                                                                                                     |
| selector.type         |         |                                                                                                                                                                                                                  |
| selector.*            |         |                                                                                                                                                                                                                  |
| interceptors          | -       | 以空格分隔的拦截器列表                                                                                                                                                                                           |
| interceptors.*        |         |                                                                                                                                                                                                                  |
| compression-type      | none    | 这可以是“none” or “deflate”。压缩类型必须与匹配Avro Source的压缩类型匹配                                                                                                                                         |
| ssl                   | false   | 将此设置为true可启用SSL加密。如果启用了SSL，则还必须通过组件级参数（见下文）或全局SSL参数（见SSL/TLS支持部分 [[flume User Guide(1.11)#^cf1b41\|ssl/tls]]）指定“keystore” 和 “keystore-password”。                                                            |
| keystore              | -       | 这是指向 Java 密钥库文件的路径。如果没有在这里指定，则将使用全局密钥库（如果全局未定义 则会发生配置错误）。                                                                                                      |
| keystore-password     | -       | Java密钥库的密码。如果未在此处指定，则将使用全局密钥库密码（如果全局未定义 则会发生配置错误）。                                                                                                                  |
| keystore-type         | JKS     | Java密钥库的类型。这可以是“JKS”或“PKCS12”。如果此处未指定，则将使用全局密钥存储类型（如果全局未定义 则默认为JKS）。                                                                                              |
| exclude-protocols     | SSLv3   | 要排除的SSL/TLS协议的空格分隔列表。除了指定的协议外，SSLv3将始终被排除在外。                                                                                                                                     |
| include-protocols     | -       | 要包含的SSL/TLS协议的空格分隔列表。启用的协议将是包含的协议，而没有排除的协议。如果包含的协议为空，则它包括所有支持的协议。                                                                                      |
| exclude-cipher-suites | -       | 要排除的以空格分隔的密码套件列表。                                                                                                                                                                               |
| include-cipher-suites | -       | 要包含的以空格分隔的密码套件列表。启用的密码套件将是包含的密码套件，而不包含排除的密码套件。如果包含的密码套件为空，则它包括所有支持的密码套件。                                                                 |
| kerberos              | false   | 设置为true可启用kerberos身份验证。在kerberos模式下，成功的身份验证需要 agent-principal 和 agent-keytab 。安全模式下的Thrift Source将只接受来自启用了kerberos并成功通过kerberos KDC身份验证的Thrift客户端的连接。 |
| agent-principal       | -       | Thrift Source用于向kerberos KDC进行身份验证的kerberos主体。                                                                                                                                                      |
| agent-keytab          | -       | Thrift Source与 agent-principal 结合使用的keytab location，用于向kerberos KDC进行身份验证。                                                                                                                      |

名为a1的 Agent 示例：
```properties
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = thrift
a1.sources.r1.channels = c1
a1.sources.r1.bind = 0.0.0.0
a1.sources.r1.port = 4141
```



#### Exec Source
Exec source 在启动时运行给定的 Unix 命令，并期望该进程不断在标准输出上产生数据（除非将属性 logStdErr 设置为 true，否则 stderr 将被简单地丢弃）。如果进程因任何原因退出，则源也会退出，并且不会再产生数据。
这意味着像 `cat [named pipe]` 或 `tail -F [file]` 这样的配置将产生预期的结果，而 `date` 可能不会。
前两个命令产生数据流，而后者产生单个事件然后退出。
必需的属性以粗体显示。

| Property Name   | Default     | Description                                                                                                  |
| --------------- | ----------- | ------------------------------------------------------------------------------------------------------------ |
| **channels**    | -           |                                                                                                              |
| **type**        | -           | 组件类型名称，需要为thrift                                                                                   |
| **command**     | -           | 要监听的主机名或IP地址                                                                                       |
| shell           | -           | 用于运行命令的shell调用。例如/bin/sh-c。仅对于依赖于外壳程序功能（如通配符、反勾号、管道等）的命令是必需的。 |
| restartThrottle | 10000       | 尝试重新启动之前等待的时间（以毫秒为单位）                                                                   |
| restart         | false       | 如果执行的cmd死了，是否应该重新启动 。                                                                       |
| logStdErr       | false       | 是否应记录命令的stderr                                                                                       |
| batchSize       | 20          | 一次读取并发送到通道的最大行数                                                                               |
| batchTimeout    | 3000        | 如果未达到缓冲区大小，则在数据被向下推送之前等待的时间（以毫秒为单位）                                       |
| selector.type   | replicating | 复制或多路传输                                                                                               |
| selector.*      |             | 取决于 selector.type的值                                                                                     |
| interceptors    | -           | 以空格分隔的拦截器列表                                                                                       |
| interceptors.*  |             |                                                                                                              |

>**Warning**
>ExecSource 和其他异步 Source 的问题在于，Source 不能保证如果将事件放入 Channel 失败，客户端就知道了。在这种情况下，数据将会丢失。例如，最常见的请求之一是类似于 `tail -F [file]` 的用例，其中应用程序将数据写入磁盘上的日志文件，而 Flume 会跟踪文件，将每行作为一个事件发送。虽然这是可能的，但存在一个明显的问题；如果 Channel 填满，Flume 就无法发送事件了，怎么办？Flume 没有办法向写日志文件的应用程序表明它需要保留日志或者由于某种原因没有发送事件。如果这听起来没有意义，你只需要知道：使用单向异步接口（如 ExecSource）时，您的应用程序永远无法保证数据已被接收！
>作为这个警告的扩展 - 为了完全清楚 - 当使用这个源时，绝对没有事件传递的保证。
>为了更强的可靠性保证.
>请考虑使用 Spooling Directory Source、Taildir Source 或通过 SDK 直接与 Flume 进行集成。

```properties
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = exec
a1.sources.r1.command = tail -F /var/log/secure
a1.sources.r1.channels = c1
```

“shell”配置用于通过命令shell（如Bash或Powershell）调用“command”。“command”作为参数传递给“shell”以执行。这允许“命令”使用shell的功能，如通配符、反勾号、管道、循环、条件等。在没有“shell”配置的情况下，将直接调用“command”。“shell”的常见值：‘/bin/sh -c’, ‘/bin/ksh -c’, ‘cmd /c’, ‘powershell -Command’等。

```properties
a1.sources.tailsource-1.type = exec
a1.sources.tailsource-1.shell = /bin/bash -c
a1.sources.tailsource-1.command = for i in /path/*.txt; do cat $i; done
```


#### JMS Source
JMS Source 从JMS目标（如队列或主题）读取消息。作为JMS应用程序，它应该与任何JMS提供程序一起工作，但仅在 ActiveMQ 中进行了测试。JMS Source 提供可配置的批大小、消息选择器、用户/传递和消息到 flume event 转换器。请注意，供应商提供的JMS jar应该使用`plugins.d` 目录（首选）、命令行上的 `–classpath` 或通过` flume-env.sh`中的`FLUME_CLASSPATH`变量包含在Flume classpath中。

所需属性以粗体显示。

| Property Name | Default | Description |
| ---- | ---- | ---- |
| **channels** | - |  |
| **type** | - | 组件类型名称，需要为 `jms` |
| **initialContextFactory** | - | 初始上下文工厂,例如:org.apache.activemq.jndi.ActiveMQInitialContextFactory |
| **connectionFactory** | - | 连接工厂应该出现的JNDI名称 |
| **providerURL** | - | JMS提供者URL |
| **destinationName** | - | 目标名称 |
| **destinationType** | - | 目标类型(队列或主题)  |
| messageSelector | - | 创建消费者时使用的消息选择器 |
| userName | - | 目的地/提供者的用户名 |
| passwordFile | - | 包含目的地/提供者密码的文件 |
| batchSize | 100 | 一批消费中的消息数量 |
| converter.type | DEFAULT | 用于将消息转换为 Flume 事件的类。 见下文。 |
| converter.* | - | 转换器属性。 |
| converter.charset | UTF-8 | 仅默认转换器。 将 JMS TextMessage 转换为字节数组时要使用的字符集。 |
| createDurableSubscription | false | 是否创建持久订阅。 持久订阅只能与destinationType 主题一起使用。 如果为 true，则必须指定“clientId”和“durableSubscriptionName” |
| clientId | - | 创建连接后立即在连接上设置 JMS 客户端标识符。 持久订阅所需 |
| durableSubscriptionName | - | 用于标识持久订阅的名称。 持久订阅所必需的。 |

##### JMS message converter

JMS source允许可插拔的转换器,尽管默认的转换器对大多数目的来说可能已经足够。默认转换器能够将Bytes、Text和Object消息转换为FlumeEvent。在所有情况下,消息中的属性都被添加为FlumeEvent的header。

- BytesMessage:
消息的字节被复制到FlumeEvent的body中。每条消息无法转换超过2GB的数据。

- TextMessage:
消息文本被转换为字节数组并复制到FlumeEvent的body中。默认使用UTF-8编码但可以配置。

- ObjectMessage:
对象被写入到包装在ObjectOutputStream中的ByteArrayOutputStream中,结果数组被复制到FlumeEvent的body中。

名为 a1 的代理示例：
```properties
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = jms
a1.sources.r1.channels = c1
a1.sources.r1.initialContextFactory = org.apache.activemq.jndi.ActiveMQInitialContextFactory
a1.sources.r1.connectionFactory = GenericConnectionFactory
a1.sources.r1.providerURL = tcp://mqserver:61616
a1.sources.r1.destinationName = BUSINESS_DATA
a1.sources.r1.destinationType = QUEUE
```


##### SSL and JMS Source

JMS客户端实现通常支持通过JSSE(Java Secure Socket Extension)定义的一些Java系统属性来配置SSL/TLS。

为Flume的JVM指定这些系统属性,JMS Source(或者更精确地说,JMS Source使用的JMS客户端实现)可以通过SSL连接到JMS服务器(当然,前提是JMS服务器也被设置为使用SSL)。

这应该适用于任何JMS提供商,并且已经在ActiveMQ、IBM MQ和Oracle WebLogic上进行了测试。

以下部分仅描述Flume一侧所需的SSL配置步骤。您可以在Flume Wiki上找到有关不同JMS提供商的服务器端设置的更详细描述以及完整的工作配置示例。

**SSL transport / server authentication:**

如果JMS服务器使用自签名证书或其证书由不可信的CA签名(例如公司自己的CA),则需要设置truststore(包含正确的证书)并传递给Flume。这可以通过全局SSL参数来完成。有关全局SSL设置的更多详细信息,请参阅SSL/TLS支持部分 [[flume User Guide(1.11)#^cf1b41|ssl/tls]]。

某些JMS提供商在使用SSL时需要特定于SSL的JNDI初始上下文工厂和/或提供商URL设置(例如,ActiveMQ使用ssl:// URL前缀而不是tcp://)。在这种情况下,必须在代理配置文件中调整source属性(initialContextFactory和/或providerURL)。

**Client certificate authentication (two-way SSL) 客户端证书认证（双向SSL）:** 

JMS Source可以通过客户端证书认证而不是通常的用户名/密码登录来向JMS服务器进行认证(当使用SSL且JMS服务器被配置为接受此类认证时)。

包含Flume用于认证的密钥的keystore需要再次通过全局SSL参数进行配置。有关全局SSL设置的详细信息,请参阅SSL/TLS支持部分 [[flume User Guide(1.11)#^cf1b41|ssl/tls]]。

keystore应该只包含一个密钥(如果存在多个密钥,则将使用第一个密钥)。密钥密码必须与keystore密码相同。

在客户端证书认证的情况下,无需在Flume代理配置文件中为JMS Source指定userName/passwordFile属性。

**请注意:**
与其他组件不同，JMS Source 没有组件级配置参数。 也没有启用 SSL 标志。 SSL 设置由 JNDI/Provider URL 设置（最终是 JMS 服务器设置）以及信任库/密钥库的存在/不存在控制。


#### Spooling Directory Source

这个source允许您通过将要摄取的文件放入磁盘上的“池”目录来摄取数据。此source将监视指定的目录以获取新文件,并在出现新文件时从中解析事件。事件解析逻辑是可插拔的。在完全读入通道后,默认情况下,通过重命名文件或删除文件或使用trackerDir来跟踪已处理文件来指示完成。

与Exec source不同,此source是可靠的,即使Flume重新启动或被杀死,也不会丢失数据。为了换取这种可靠性,只能将不可变的、唯一命名的文件放入池目录中。Flume会试图检测这些问题条件,如果违反这些条件,会发出很大的错误提示:

1. 如果在放入池目录后文件被写入,Flume将向其日志文件打印错误并停止处理。
2. 如果后来重用了文件名,Flume将向其日志文件打印错误并停止处理。

为了避免上述问题,当日志文件被移动到池目录中时,可以考虑在日志文件名中添加唯一标识符(如时间戳)。

尽管此source提供了可靠性保证,但如果某些下游失败发生,事件可能会被复制。这与其他Flume组件提供的保证一致。


|Property Name|Default|描述|
|---|---|---|
|**channels**|–||
|**type**|–|组件类型名称,需要为spooldir|
|**spoolDir**|–|读取文件目录路径|
|fileSuffix|.COMPLETED|完全摄取文件的后缀|
|deletePolicy|never|完成文件删除策略:never 或 immediate|
|fileHeader|false|是否添加头保存完整文件名|
|fileHeaderKey|file|文件名添加到 event header 的键|
|basenameHeader|false|是否添加头保存文件名|
|basenameHeaderKey|basename|文件名添加到 event header 的键|
|includePattern|^.*$|指定包含哪些文件正则表达式。可以与ignorePattern一起使用。如果文件同时匹配ignorePattern和includePattern则忽略。|
|ignorePattern|^$|指定忽略哪些文件的正则表达式。可以与includePattern一起使用。如果文件同时匹配ignorePattern和includePattern则忽略。|
|trackerDir|.flumespool|存储文件处理元数据的目录。如果不是绝对路径,则相对于spoolDir。|
|trackingPolicy|rename|定义文件处理追踪策略。可以是“rename”或“tracker_dir”。仅在deletePolicy为“never”时有效。“rename” - 处理后根据fileSuffix重命名文件。“tracker_dir” - 文件不重命名,在trackerDir中创建一个空文件。新tracker文件名基于摄取文件名加上fileSuffix。|
|consumeOrder|oldest|池目录中文件消费顺序:oldest最旧,youngest最新,random随机。oldest和youngest基于最后修改时间比较。random完全随机选择。oldest和youngest需全目录扫描选择最旧/最新文件,如果大量文件效率低;random可能导致旧文件被延后消费。|
|pollDelay|500|轮询新文件的延迟(毫秒)|
|recursiveDirectorySearch|false|是否监控子目录的新文件|
|maxBackoff|4000|写入channel失败的最大退避时间(毫秒)。从低backoff指数增加到配置值。|
|batchSize|100|批量传输到channel的粒度|
|inputCharset|UTF-8|处理文本文件时使用的字符集|
|decodeErrorPolicy|FAIL|发现不可解码字符时的处理策略。FAIL:抛异常失败;REPLACE:替换为特殊未知字符;IGNORE:忽略|
|deserializer|LINE|解析文件到事件的反序列化器。默认逐行解析。必须实现EventDeserializer.Builder。|
|deserializer.*||因反序列化器而异|
|bufferMaxLines|–|(废弃)现在忽略|
|bufferMaxLineLength|5000|(废弃)替代为deserializer.maxLineLength|
|selector.type|replicating|复制或复用|
|selector.*||依赖selector.type值|
|interceptors|–|空格分隔拦截器列表|
|interceptors.*|||

名为 a1 的代理示例：

```properties
a1.channels = ch-1
a1.sources = src-1

a1.sources.src-1.type = spooldir
a1.sources.src-1.channels = ch-1
a1.sources.src-1.spoolDir = /var/log/apache/flumeSpool
a1.sources.src-1.fileHeader = true
```

##### Event Deserializers

Flume 附带以下事件反序列化器。

###### LINE

这个反序列化器针对每行文本输入生成一个事件。

| Property Name | Default | Description |
| ---- | ---- | ---- |
| deserializer.maxLineLength | 2048 | 单个事件中包含的最大字符数。 如果一行超过此长度，则会被截断，并且该行中剩余的字符将出现在后续事件中。 |
| deserializer.outputCharset | UTF-8 | 用于对放入 channel 的事件进行编码的字符集。 |
###### AVRO

这个反序列化器能够读取Avro容器文件,并对文件中的每条Avro记录生成一个事件。每个事件都带有一个标头,指示使用的模式。事件的body是二进制的Avro记录数据,不包括schema或容器文件中的其他元素。

注意,如果spool目录源必须重试将这些事件之一放到通道上(例如,因为通道已满),那么它将重置并从最新的Avro容器文件同步点重试。为了减少这种失败场景中的潜在事件重复,可以在Avro输入文件中更频繁地写入同步标记。

|Property Name|Default|Description|
|---|---|---|
|deserializer.schemaType|HASH|架构的表示方式。 默认情况下，或者当指定 HASH 值时，Avro 架构会进行哈希处理，并且哈希值存储在事件标头“flume.avro.schema.hash”中的每个事件中。 如果指定了 LITERAL，则 JSON 编码的架构本身存储在事件标头“flume.avro.schema.literal”中的每个事件中。 与 HASH 模式相比，使用 Literal 模式的效率相对较低。 |

###### BlobDeserializer

此反序列化器针对每个事件读取一个二进制大对象 (BLOB)，通常每个文件读取一个 BLOB。 例如 PDF 或 JPG 文件。 请注意，此方法不适合非常大的对象，因为整个 BLOB 都缓冲在 RAM 中。

|Property Name|Default|Description|
|---|---|---|
|**deserializer**|–|这个类的完全限定类名: org.apache.flume.sink.solr.morphline.BlobDeserializer$Builder |
|deserializer.maxBlobLength|100000000|针对给定请求读取和缓冲的最大字节数 |


#### Taildir Source

> **注意 此源作为预览功能提供。 它不适用于 Windows。**

监视指定的文件,在检测到每份文件中追加了新行时,几乎实时地对其进行尾部跟踪。如果新行正在被写入,此source将重试读取以等待写入完成。

此source是可靠的,即使尾部文件的轮换也不会丢失数据。它会定期以JSON格式将每个文件的最后读取位置写入指定的位置文件。如果Flume停止或由于某些原因关闭,它可以从现有位置文件中写入的位置重新开始尾部跟踪。

在其他用例中,此source还可以使用给定的位置文件从每个文件的任意位置开始尾部跟踪。如果在指定路径上没有位置文件,它将默认从每个文件的第一行开始跟踪。

文件将按修改时间顺序进行消费。修改时间最早的文件将首先被消费。

此source不会重命名、删除或修改被跟踪的文件。当前此source不支持尾部跟踪二进制文件。它逐行读取文本文件。


|Property Name|Default|描述|
|---|---|---|
|**channels**|–||
|**type**|–|组件类型名称,需要为TAILDIR|
|**filegroups**|–|空格分隔的文件组列表。每个文件组表示一组要tailed的文件|
|**filegroups.\<filegroupName>**|–|文件组的绝对路径。文件名只能使用正则表达式(而不是文件系统通配符)|
|positionFile|~/.flume/taildir_position.json|记录每个tailed文件inode、路径和最后位置的JSON格式文件|
|headers.\<filegroupName>.\<headerKey>|–|设置具有header key的header值。每个file group可以指定多个header|
|byteOffsetHeader|false|是否在名为“byteoffset”的标头中添加tailed行的字节偏移量|
|skipToEnd|false|如果位置文件中没有写入,是否跳过到EOF的位置|
|idleTimeout|120000|关闭不活动文件的时间(毫秒)。如果该文件有新行追加,源会自动重新打开|
|writePosInterval|3000|写入每个文件最后位置到位置文件的时间间隔(毫秒)|
|batchSize|100|每次读取和发送到channel的最大行数。默认通常就好|
|maxBatchCount|Long.MAX_VALUE|从同一文件中连续读取的批处理数的控制。如果源正在跟踪多个文件,并且其中一个以更快的速率写入,则可能会阻止处理其他文件,因为繁忙文件的读取将无限循环。在这种情况下,降低此值|
|backoffSleepIncrement|1000|在上一次尝试未找到新数据时,重试轮询新数据之前的延迟递增时间|
|maxBackoffSleep|5000|在上一次尝试未找到新数据时,每次重试轮询新数据之间的最大延迟时间|
|cachePatternMatching|true|列出目录和应用文件名正则表达式匹配可能非常耗时,如果目录中包含数千个文件。缓存匹配文件的列表可以提高性能。消费文件的顺序也会被缓存。要求文件系统以至少1秒的粒度跟踪修改时间|
|fileHeader|false|是否添加一个标头存储完整文件名|
|fileHeaderKey|file|追加绝对文件名到事件标头时使用的标头键|

名为 a1 的代理示例：

```properties
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = TAILDIR
a1.sources.r1.channels = c1
a1.sources.r1.positionFile = /var/log/flume/taildir_position.json
a1.sources.r1.filegroups = f1 f2
a1.sources.r1.filegroups.f1 = /var/log/test1/example.log
a1.sources.r1.headers.f1.headerKey1 = value1
a1.sources.r1.filegroups.f2 = /var/log/test2/.*log.*
a1.sources.r1.headers.f2.headerKey1 = value2
a1.sources.r1.headers.f2.headerKey2 = value2-2
a1.sources.r1.fileHeader = true
a1.sources.ri.maxBatchCount = 1000
```


#### Twitter 1% firehose Source (experimental)

> **警告: 此源是高度实验性的，可能会在 Flume 的次要版本之间发生变化。 使用风险自负。**

通过 Streaming API 连接到 1% 样本 twitter firehose 的实验源，持续下载推文，将其转换为 Avro 格式并将 Avro 事件发送到下游 Flume 接收器。 需要 Twitter 开发者帐户的消费者和访问令牌和秘密。 必需的属性以粗体显示。

| Property Name | Default | Description |  |
| ---- | ---- | ---- | ---- |
| **channels** | – |  |  |
| **type** | – | 组件类型名称，需要是 `org.apache.flume.source.twitter.TwitterSource` |  |
| **consumerKey** | – | OAuth consumer key |  |
| **consumerSecret** | – | OAuth consumer secret |  |
| **accessToken** | – | OAuth access token |  |
| **accessTokenSecret** | – | OAuth token secret |  |
| maxBatchSize | 1000 | 单个批次中放入的 Twitter 消息的最大数量 |  |
| maxBatchDurationMillis | 1000 | 关闭批次之前等待的最大毫秒数 |  |


名为 a1 的代理示例：
```properties
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = org.apache.flume.source.twitter.TwitterSource
a1.sources.r1.channels = c1
a1.sources.r1.consumerKey = YOUR_TWITTER_CONSUMER_KEY
a1.sources.r1.consumerSecret = YOUR_TWITTER_CONSUMER_SECRET
a1.sources.r1.accessToken = YOUR_TWITTER_ACCESS_TOKEN
a1.sources.r1.accessTokenSecret = YOUR_TWITTER_ACCESS_TOKEN_SECRET
a1.sources.r1.maxBatchSize = 10
a1.sources.r1.maxBatchDurationMillis = 200

```


#### Kafka Source

Kafka Source 是一个 Apache Kafka 消费者，它从 Kafka 主题读取消息。 如果您有多个 Kafka Source 正在运行，您可以使用相同的 Consumer Group 配置它们，这样每个源都会读取一组唯一的主题分区。 目前支持 Kafka 服务器版本 0.10.1.0 或更高版本。 测试已完成至 2.0.1，这是发布时的最高可用版本。

|属性名称|默认值|描述|
|---|---|---|
|**channels**|–||
|**type**|–|组件类型名称,需要为 org.apache.flume.source.kafka.KafkaSource|
|**kafka.bootstrap.servers**|–|源使用的Kafka集群中代理的列表|
|kafka.consumer.group.id|flume|消费者组的唯一标识。在多个源或代理中设置相同ID表示它们属于同一个消费者组|
|**kafka.topics**|–|Kafka消费者将读取消息的主题列表,以逗号分隔。|
|**kafka.topics.regex**|–|定义源订阅的主题集的正则表达式。此属性的优先级高于kafka.topics,如果存在则会覆盖kafka.topics。|
|batchSize|1000|一次批量写到Channel中的消息的最大数量|
|batchDurationMillis|1000|在批处理被写入Channel之前的最大时间(毫秒)。无论大小还是时间达到的先决条件,批处理都将被写入。|
|backoffSleepIncrement|1000|当Kafka主题为空时会触发的初始和增量等待时间。等待时间会减少对空Kafka主题的激进ping。对于摄取用例,1秒是理想的,但对于带拦截器的低延迟操作,可能需要较低的值。|
|maxBackoffSleep|5000|当Kafka主题为空时触发的最大等待时间。对于摄取用例,5秒是理想的,但对于带拦截器的低延迟操作,可能需要较低的值。|
|useFlumeEventFormat|false|默认情况下,事件作为字节直接从Kafka主题获取到事件体中。设置为true可将事件读取为Flume Avro二进制格式。与KafkaSink上的相同属性结合使用,或与Kafka Channel上的parseAsFlumeEvent属性结合使用,这将保留在生产端发送的任何Flume标头。|
|setTopicHeader|true|设置为true时,会将检索到的消息的主题存储到标头中,由topicHeader属性定义。|
|topicHeader|topic|如果设置了setTopicHeader属性,则定义用于存储接收消息的主题名称的标头的名称。 如果与Kafka Sink topicHeader属性组合,应小心以避免消息被发送回同一主题造成循环。|
|timestampHeader|–|如果存在,Kafka消息时间戳值将被复制到指定的Flume标头名称。|
|header.NAME|–|用于标识哪些标头应该从Kafka消息添加为Flume标头。NAME的值应与Flume标头名称匹配,值应为要用作Kafka标头名称的标头的名称。|
|kafka.consumer.security.protocol|PLAINTEXT|如果使用某种安全级别写入Kafka,请设置为SASL_PLAINTEXT、SASL_SSL或SSL。有关安全设置的其他信息,请参见下文。|
|_更多消费者安全属性_||如果使用SASL_PLAINTEXT、SASL_SSL或SSL,请参考 [Kafka security]([https://kafka.apache.org/documentation.html#security](https://kafka.apache.org/documentation.html#security)) 以获取在消费者上需要设置的其他属性。|
|其他Kafka消费者属性|–|这些属性用于配置Kafka消费者。支持的任何消费者属性 都可以使用。唯一的要求是使用前缀 kafka.consumer 为属性名称添加前缀。 例如:kafka.consumer.auto.offset.reset|

> 注意： Kafka Source 覆盖两个 Kafka Consumer 参数：auto.commit.enable 由源设置为“false”，并且每个批次都会提交。 Kafka源保证至少一次消息检索策略。 源启动时可能会出现重复项。 Kafka Source 还提供了 key.deserializer(org.apache.kafka.common.serialization.StringSerializer) 和 value.deserializer(org.apache.kafka.common.serialization.ByteArraySerializer) 的默认值。 不建议修改这些参数。


已弃用的属性

|Property Name|Default|Description|
|---|---|---|
|topic|–|使用 kafka.topics |
|groupId|flume|使用 kafka.consumer.group.id |
|zookeeperConnect|–|自 0.9.x 起，Kafka 消费者客户端不再支持。 使用kafka.bootstrap.servers与Kafka集群建立连接 |
|migrateZookeeperOffsets|true|当没有找到Kafka存储的偏移量时，在Zookeeper中查找偏移量并将其提交给Kafka。 为了支持从旧版本 Flume 无缝迁移 Kafka 客户端，这应该是正确的。 迁移后，可以将其设置为 false，但通常不需要这样做。 如果没有找到 Zookeeper 偏移量，Kafka 配置 kafka.consumer.auto.offset.reset 定义如何处理偏移量。 有关详细信息，请查看 [Kafka 文档](https://kafka.apache.org/documentation.html#consumerconfigs)<br>Dāng méiyǒu zhǎodào Kafka cúnchú de piān yí liàng shí, zài Zookeeper zhōng cházhǎo pi |


通过逗号分隔的主题列表进行主题订阅的示例
```properties
tier1.sources.source1.type = org.apache.flume.source.kafka.KafkaSource
tier1.sources.source1.channels = channel1
tier1.sources.source1.batchSize = 5000
tier1.sources.source1.batchDurationMillis = 2000
tier1.sources.source1.kafka.bootstrap.servers = localhost:9092
tier1.sources.source1.kafka.topics = test1, test2
tier1.sources.source1.kafka.consumer.group.id = custom.g.id
```

通过正则表达式订阅主题的示例

```properties
tier1.sources.source1.type = org.apache.flume.source.kafka.KafkaSource
tier1.sources.source1.channels = channel1
tier1.sources.source1.kafka.bootstrap.servers = localhost:9092
tier1.sources.source1.kafka.topics.regex = ^topic[0-9]$
# the default kafka.consumer.group.id=flume is used
```


**Security and Kafka Source:**

Flume 和 Kafka 之间的通信通道支持安全身份验证和数据加密。 对于安全身份验证，可以从 Kafka 版本 0.9.0 开始使用 SASL/GSSAPI (Kerberos V5) 或 SSL（即使参数名为 SSL，但实际协议是 TLS 实现）。

目前,数据加密仅由SSL/TLS提供。
将kafka.consumer.security.protocol设置为以下任何值意味着:

- **SASL_PLAINTEXT** - Kerberos或明文认证,没有数据加密
- **SASL_SSL** - Kerberos或明文认证,数据加密
- **SSL** - 基于TLS的加密,可选认证。


> **警告 启用 SSL 时，性能会下降，其幅度取决于 CPU 类型和 JVM 实现。 参考：Kafka安全概述和跟踪此问题的jira：[KAFKA-2561](https://issues.apache.org/jira/browse/KAFKA-2561)**


**TLS and Kafka Source:**

请阅读配置 [Kafka 客户端 SSL 中描述的步骤](https://kafka.apache.org/documentation/#security_configclients)，了解用于微调的其他配置设置，例如以下任何一项：安全提供程序、密码套件、启用的协议、信任库或密钥库类型。

服务器端身份验证和数据加密的示例配置。
```properties
a1.sources.source1.type = org.apache.flume.source.kafka.KafkaSource
a1.sources.source1.kafka.bootstrap.servers = kafka-1:9093,kafka-2:9093,kafka-3:9093
a1.sources.source1.kafka.topics = mytopic
a1.sources.source1.kafka.consumer.group.id = flume-consumer
a1.sources.source1.kafka.consumer.security.protocol = SSL
# optional, the global truststore can be used alternatively
a1.sources.source1.kafka.consumer.ssl.truststore.location=/path/to/truststore.jks
a1.sources.source1.kafka.consumer.ssl.truststore.password=<password to access the truststore>
```

此处指定信任库是可选的，可以使用全局信任库。 有关全局 SSL 设置的更多详细信息，请参阅 SSL/TLS 支持部分。

注意：默认情况下，未定义属性 ssl.endpoint.identification.algorithm，因此不执行主机名验证。 为了启用主机名验证，请设置以下属性
```properties
a1.sources.source1.kafka.consumer.ssl.endpoint.identification.algorithm=HTTPS
```

启用后，客户端将根据以下两个字段之一验证服务器的完全限定域名 (FQDN)：

- 通用名称（中文） https://tools.ietf.org/html/rfc6125#section-2.3
- 主题备用名称 (SAN) https://tools.ietf.org/html/rfc5280#section-4.2.1.6
如果还需要客户端身份验证，则还需要将以下内容添加到 Flume 代理配置中，或者可以使用全局 SSL 设置（请参阅 SSL/TLS 支持部分 [[flume User Guide(1.11)#^cf1b41|ssl/tls]]）。 每个 Flume 代理都必须拥有其客户端证书，该证书必须受到 Kafka 代理的单独信任或通过其签名链信任。 常见的示例是由单个根 CA 签署每个客户端证书，而该根 CA 又受到 Kafka 代理的信任。

```properties
# optional, the global keystore can be used alternatively
a1.sources.source1.kafka.consumer.ssl.keystore.location=/path/to/client.keystore.jks
a1.sources.source1.kafka.consumer.ssl.keystore.password=<password to access the keystore>
```

如果密钥库和密钥使用不同的密码保护，则 ssl.key.password 属性将为两个消费者密钥库提供所需的附加秘密：
```properties
a1.sources.source1.kafka.consumer.ssl.key.password=<password to access the key>
```

**Kerberos and Kafka Source:**

要将 Kafka Source 与受 Kerberos 保护的 Kafka 集群一起使用，请为消费者设置上面提到的 Consumer.security.protocol 属性。 与 Kafka 代理一起使用的 Kerberos 密钥表和主体在 JAAS 文件的“KafkaClient”部分中指定。 “客户端”部分描述了 Zookeeper 连接（如果需要）。 有关 JAAS 文件内容的信息，请参阅 [Kafka 文档](https://kafka.apache.org/documentation.html#security_sasl_clientconfig)。 该 JAAS 文件的位置以及可选的系统范围 kerberos 配置可以通过 Flume-env.sh 中的 JAVA_OPTS 指定：
```shell
JAVA_OPTS="$JAVA_OPTS -Djava.security.krb5.conf=/path/to/krb5.conf"
JAVA_OPTS="$JAVA_OPTS -Djava.security.auth.login.config=/path/to/flume_jaas.conf"
```

使用 SASL_PLAINTEXT 的安全配置示例：

```properties
a1.sources.source1.type = org.apache.flume.source.kafka.KafkaSource
a1.sources.source1.kafka.bootstrap.servers = kafka-1:9093,kafka-2:9093,kafka-3:9093
a1.sources.source1.kafka.topics = mytopic
a1.sources.source1.kafka.consumer.group.id = flume-consumer
a1.sources.source1.kafka.consumer.security.protocol = SASL_PLAINTEXT
a1.sources.source1.kafka.consumer.sasl.mechanism = GSSAPI
a1.sources.source1.kafka.consumer.sasl.kerberos.service.name = kafka
```

使用 SASL_SSL 的安全配置示例：
```properties
a1.sources.source1.type = org.apache.flume.source.kafka.KafkaSource
a1.sources.source1.kafka.bootstrap.servers = kafka-1:9093,kafka-2:9093,kafka-3:9093
a1.sources.source1.kafka.topics = mytopic
a1.sources.source1.kafka.consumer.group.id = flume-consumer
a1.sources.source1.kafka.consumer.security.protocol = SASL_SSL
a1.sources.source1.kafka.consumer.sasl.mechanism = GSSAPI
a1.sources.source1.kafka.consumer.sasl.kerberos.service.name = kafka
# optional, the global truststore can be used alternatively
a1.sources.source1.kafka.consumer.ssl.truststore.location=/path/to/truststore.jks
a1.sources.source1.kafka.consumer.ssl.truststore.password=<password to access the truststore>
```

示例 JAAS 文件。 有关其内容的参考，请参阅 [SASL 配置的 Kafka 文档](https://kafka.apache.org/documentation/#security_sasl_clientconfig)中所需身份验证机制 (GSSAPI/PLAIN) 的客户端配置部分。 由于Kafka Source也可能连接到Zookeeper进行偏移迁移，因此本示例中还添加了“Client”部分。 除非您需要偏移迁移，或者您需要此部分用于其他安全组件，否则不需要此部分。 另请确保 Flume 进程的操作系统用户具有 jaas 和 keytab 文件的读取权限。

```
Client {
  com.sun.security.auth.module.Krb5LoginModule required
  useKeyTab=true
  storeKey=true
  keyTab="/path/to/keytabs/flume.keytab"
  principal="flume/flumehost1.example.com@YOURKERBEROSREALM";
};

KafkaClient {
  com.sun.security.auth.module.Krb5LoginModule required
  useKeyTab=true
  storeKey=true
  keyTab="/path/to/keytabs/flume.keytab"
  principal="flume/flumehost1.example.com@YOURKERBEROSREALM";
};
```


#### NetCat TCP Source

类似 netcat 的源，监听给定端口并将每一行文本转换为一个事件。 其作用类似于 
`nc -k -l [host] [port]`。 
换句话说，它打开指定的端口并监听数据。 期望提供的数据是换行分隔的文本。 每行文本都会转化为 Flume 事件并通过连接的通道发送。
必需的属性以粗体显示

|属性名称|默认值|描述|
|---|---|---|
|**channels**|–||
|**type**|–|组件类型名称,需要为 netcat|
|**bind**|–|要绑定的主机名或 IP 地址|
|**port**|–|要绑定的端口号|
|max-line-length|512|每条事件体的最大行长度(以字节为单位)|
|ack-every-event|true|对收到的每条事件响应“OK”|
|selector.type|replicating|replicating 或 multiplexing|
|selector.*||依赖于 selector.type 值|
|interceptors|–|空格分隔的拦截器列表|
|interceptors.*|||

名为 a1 的代理示例：

```properties
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = netcat
a1.sources.r1.bind = 0.0.0.0
a1.sources.r1.port = 6666
a1.sources.r1.channels = c1
```


#### NetCat UDP Source

根据原始 Netcat (TCP) 源，该源侦听给定端口并将每行文本转换为事件并通过连接的通道发送。 行为就像 `nc -u -k -l [host] [port]`
必需的属性以粗体显示

| 属性名称 | 默认值 | 描述 |
| ---- | ---- | ---- |
| **channels** | – |  |
| **type** | – | 组件类型名称,需要为 `netcatudp` |
| **bind** | – | 要绑定的主机名或 IP 地址 |
| **port** | – | 要绑定的端口号 |
| remoteAddressHeader | – |  |
| selector.type | replicating | replicating 或 multiplexing |
| selector.* |  | 依赖于 selector.type 值 |
| interceptors | – | 空格分隔的拦截器列表 |
| interceptors.* |  |  |

名为 a1 的代理示例：
```properties
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = netcatudp
a1.sources.r1.bind = 0.0.0.0
a1.sources.r1.port = 6666
a1.sources.r1.channels = c1
```

#### Sequence Generator Source

一个简单的序列生成器，它使用从 0 开始、以 1 递增并在totalEvents 处停止的计数器连续生成事件。 当无法将事件发送到通道时重试。 主要用于测试。 在重试期间，它会保持重试消息的正文与之前相同，以便在目的地进行重复数据删除之后，唯一事件的数量预计等于指定的totalEvents。 必需的属性以粗体显示。

|属性名称|默认值|描述|
|---|---|---|
|**channels**|–||
|**type**|–|组件类型名称,需要为 seq|
|selector.type||replicating 或 multiplexing|
|selector.*|replicating|取决于 selector.type 值|
|interceptors|–|空格分隔的拦截器列表|
|interceptors.*|||
|batchSize|1|每次请求循环尝试处理的事件数。|
|totalEvents|Long.MAX_VALUE|源发送的唯一事件数。|

名为 a1 的代理示例：
```properties
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = seq
a1.sources.r1.channels = c1
```

#### Syslog Sources

读取系统日志数据并生成 Flume 事件。 UDP 源将整个消息视为单个事件。 TCP 源为每个由换行符 (‘n’) 分隔的字符串创建一个新事件。
必需的属性以粗体显示。
##### Syslog TCP Source

原始的、经过验证的 syslog TCP 源。

|属性名称|默认值|描述|
|---|---|---|
|**channels**|–||
|**type**|–|组件类型名称,需要为 syslogtcp|
|**host**|–|要绑定的主机名或 IP 地址|
|**port**|–|要绑定的端口号|
|eventSize|2500|单个事件行的最大大小(以字节为单位)|
|keepFields|none|将此设置为“all”将在事件正文中保留优先级、时间戳和主机名。也允许以空格分隔的字段列表。当前,可以包含以下字段:priority、version、timestamp、hostname。值“true”和“false”已弃用,改用“all”和“none”。 |
|clientIPHeader|–|如果指定,客户端的 IP 地址将使用此处指定的标头名称存储在每个事件的标头中。这允许拦截器和通道选择器根据客户端的 IP 地址自定义路由逻辑。不要在这里使用标准的 Syslog 标头名称(如 _host_),因为事件标头在那种情况下会被覆盖。|
|clientHostnameHeader|–|如果指定,客户端的主机名将使用此处指定的标头名称存储在每个事件的标头中。这允许拦截器和通道选择器根据客户端的主机名自定义路由逻辑。检索主机名可能涉及名称服务反向查找,这可能会影响性能。不要在这里使用标准的 Syslog 标头名称(如 _host_),因为事件标头在那种情况下会被覆盖。|
|selector.type||replicating 或 multiplexing|
|selector.*|replicating|取决于 selector.type 值|
|interceptors|–|空格分隔的拦截器列表|
|interceptors.*|||
|ssl|false|将此设置为 true 以启用 SSL 加密。如果启用 SSL,还必须通过组件级参数(参见下文)或全局 SSL 参数(参见 [SSL/TLS support](https://flume.apache.org/releases/content/1.11.0/FlumeUserGuide.html#ssl-tls-support)) 部分)指定“keystore”和“keystore-password”。 |
|keystore|–|这是 Java 密钥库文件的路径。如果在这里未指定,则将使用全局密钥库(如果已定义,否则将产生配置错误)。|
|keystore-password|–| Java 密钥库的密码。如果在此未指定,则将使用全局密钥库密码(如果已定义,否则会产生配置错误)。|
|keystore-type|JKS| Java 密钥库的类型。可以是 “JKS” 或 “PKCS12”。如果在此处未指定,则将使用全局密钥库类型(如果已定义,否则默认为 JKS)。|
|exclude-protocols|SSLv3|要排除的 SSL/TLS 协议的以空格分隔的列表。除了指定的协议之外,还将始终排除 SSLv3。|
|include-protocols|–|要包括的 SSL/TLS 协议的以空格分隔的列表。启用的协议将是不含排除协议的包含协议。如果 include-protocols 为空,它将包含每个支持的协议。|
|exclude-cipher-suites|–|要排除的密码套件的以空格分隔的列表。|
|include-cipher-suites|–|要包含的密码组合的以空格分隔的列表。启用的密码组合将是不含排除的密码组合的包含密码组合。如果 include-cipher-suites 为空,它将包含每个支持的密码组合。|

例如，名为 a1 的代理的 syslog TCP source：
```properties
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = syslogtcp
a1.sources.r1.port = 5140
a1.sources.r1.host = localhost
a1.sources.r1.channels = c1
```


##### Multiport Syslog TCP Source

这是 Syslog TCP 源的更新、更快、支持多端口的版本。 请注意，端口配置设置已替换端口。 多端口功能意味着它可以高效地同时侦听多个端口。 该源使用 Apache Mina 库来执行此操作。 提供对 RFC-3164 和许多常见 RFC-5424 格式消息的支持。 还提供配置每个端口使用的字符集的功能。

|属性名称|默认值|描述|
|---|---|---|
|**channels**|–||
|**type**|–|组件类型名称,需要为 multiport_syslogtcp|
|**host**|–|要绑定的主机名或 IP 地址|
|**ports**|–|要绑定的一个或多个端口的空格分隔列表|
|eventSize|2500|单个事件行的最大大小(以字节为单位)|
|keepFields|none|将此设置为“all”将在事件正文中保留优先级、时间戳和主机名。也允许以空格分隔的字段列表。 当前可以包含以下字段:priority、version、timestamp、hostname。值“true”和“false”已弃用,改用“all”和“none”。|
|portHeader|–|如果指定,端口号将使用此处指定的标头名称存储在每个事件的标头中。这允许拦截器和通道选择器根据传入端口自定义路由逻辑。|
|clientIPHeader|–|如果指定,客户端的IP地址将使用此处指定的标头名称存储在每个事件的标头中。这允许拦截器和通道选择器根据客户端的IP地址自定义路由逻辑。不要在这里使用标准的Syslog标头名称(如 \_host\_) 因为事件标头在那种情况下会被覆盖。 |
|clientHostnameHeader|–|如果指定,客户端的主机名将使用此处指定的标头名称存储在每个事件的标头中。这允许拦截器和通道选择器根据客户端的主机名自定义路由逻辑。检索主机名可能涉及名称服务反向查找,这可能会影响性能。不要在这里使用标准的Syslog标头名称(如 \_host\_)因为事件标头在那种情况下会被覆盖。 |
|charset.default|UTF-8|解析syslog事件为字符串时使用的默认字符集。|
|charset.port.\<port> |–|可以对每个端口配置字符集。|
|batchSize|100|每次请求循环尝试处理的最大事件数。使用默认值通常就好。|
|readBufferSize|1024|内部Mina读取缓冲区的大小。提供性能优化。使用默认值通常就好。|
|numProcessors|(auto-detected)|用于处理消息的系统上可用的处理器数。默认情况下,使用Java运行时API自动检测CPU数。Mina将为每个检测到的CPU生成2个请求处理线程,这通常是合理的。|
|selector.type|replicating|replicating、multiplexing或custom|
|selector.*|–|依赖于selector.type的值|
|interceptors|–|空格分隔的拦截器列表。|
|interceptors.*|||
|ssl|false|将此设置为true可启用SSL加密。 如果启用SSL,还必须通过组件级参数(参见下文)或全局SSL参数(请参阅[SSL/TLS支持](https://flume.apache.org/releases/content/1.11.0/FlumeUserGuide.html#ssl-tls-support)部分)指定“keystore”和“keystore-password”。|
|keystore|–|这是Java密钥库文件的路径。如果在这里未指定,则将使用全局密钥库(如果已定义,否则将产生配置错误)。|
|keystore-password|–|Java密钥库的密码。如果在此未指定,则将使用全局密钥库密码(如果已定义,否则会产生配置错误)。|
|keystore-type|JKS|Java密钥库的类型。可以是“JKS”或“PKCS12”。如果在此处未指定,则将使用全局密钥库类型(如果已定义,否则默认为JKS)。|
|exclude-protocols|SSLv3|要排除的SSL/TLS协议的以空格分隔的列表。除了指定的协议之外,还将始终排除SSLv3。|
|include-protocols|–|要包含的SSL/TLS协议的以空格分隔的列表。启用的协议将是不含排除协议的包含协议。如果include-protocols为空,它将包含每个支持的协议。|
|exclude-cipher-suites|–|要排除的密码套件的以空格分隔的列表。|
|include-cipher-suites|–|要包含的密码组合的以空格分隔的列表。启用的密码组合将是不含排除的密码组合的包含密码组合。如果include-cipher-suites为空,它将包含每个支持的密码组合。|

##### Syslog UDP Source

|属性名称|默认值|描述|
|---|---|---|
|**channels**|–||
|**type**|–|组件类型名称,需要为 syslogudp|
|**host**|–|要绑定的主机名或 IP 地址|
|**port**|–|要绑定的端口号|
|keepFields|false|将此设置为 true 将在事件正文中保留优先级、时间戳和主机名。|
|clientIPHeader|–|如果指定,客户端的 IP 地址将使用此处指定的标头名称存储在每个事件的标头中。这允许拦截器和通道选择器根据客户端的 IP 地址自定义路由逻辑。不要在这里使用标准的 Syslog 标头名称(如 \_host\_),因为事件标头在那种情况下会被覆盖。 |
|clientHostnameHeader|–|如果指定,客户端的主机名将使用此处指定的标头名称存储在每个事件的标头中。这允许拦截器和通道选择器根据客户端的主机名自定义路由逻辑。检索主机名可能涉及名称服务反向查找,这可能会影响性能。不要在这里使用标准的 Syslog 标头名称(如 \_host\_),因为事件标头在那种情况下会被覆盖。 |
|selector.type||replicating 或 multiplexing|
|selector.*|replicating|取决于 selector.type 值|
|interceptors|–|空格分隔的拦截器列表|
|interceptors.*|||

例如，名为 a1 的代理的 syslog UDP source：
```properties
a1.sources = r1
a1.channels = c1
a1.sources.r1.type = syslogudp
a1.sources.r1.port = 5140
a1.sources.r1.host = localhost
a1.sources.r1.channels = c1
```


#### HTTP Source

通过 HTTP POST 和 GET 接受 Flume 事件的源。 GET 只能用于实验。 HTTP 请求通过可插入“handler”转换为 Flume 事件，该处理程序必须实现 HTTPSourceHandler 接口。 该处理程序接受 HttpServletRequest 并返回 Flume 事件列表。 从一个 Http 请求处理的所有事件都会在一个事务中提交到通道，从而提高文件通道等通道的效率。 如果处理程序引发异常，该源将返回 HTTP 状态 400。如果通道已满，或者源无法将事件附加到通道，则源将返回 HTTP 503 - 暂时不可用状态。

在一个 post 请求中发送的所有事件都被视为一批并在一笔事务中插入到通道中。

该源基于 Jetty 9.4，并提供设置其他 Jetty 特定参数的能力，这些参数将直接传递到 Jetty 组件。

