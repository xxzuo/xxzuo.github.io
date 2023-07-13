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
| ssl                   | false   | 将此设置为true可启用SSL加密。如果启用了SSL，则还必须通过组件级参数（见下文）或全局SSL参数（见SSL/TLS支持部分）指定“keystore” 和 “keystore-password”。                                                            |
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

| Property Name             | Default | Description                                                                                                  |
| ------------------------- | ------- | ------------------------------------------------------------------------------------------------------------ |
| **channels**              | -       |                                                                                                              |
| **type**                  | -       | 组件类型名称，需要为thrift                                                                                   |
| **initialContextFactory** | -       | 要监听的主机名或IP地址                                                                                       |
| **connectionFactory**     | -       | 用于运行命令的shell调用。例如/bin/sh-c。仅对于依赖于外壳程序功能（如通配符、反勾号、管道等）的命令是必需的。 |
| **providerURL**           | 10000   | 尝试重新启动之前等待的时间（以毫秒为单位）                                                                   |
| **destinationName**       |         |                                                                                                              |
| **destinationType**       |         |                                                                                                              |
|                           |         |                                                                                                              |
