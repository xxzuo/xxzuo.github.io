---
title: flume自定义拦截器(Interceptor)
author: xxzuo
tags:
  - flume
categories:
  - flume
date: 2022-12-08 23:09:12
---



自定义Interceptor是Flume中一个非常有用的功能，它允许用户在Flume中添加自己的拦截器，以实现自定义的日志处理逻辑。

要创建一个自定义的Interceptor，需要实现Flume的`Interceptor`接口，并实现接口中定义的所有方法。然后，可以在Flume的配置文件中添加该拦截器，并指定它的位置。

拦截器将在Flume中的数据流中执行，并可以对数据进行处理、修改、过滤或转换。拦截器的处理逻辑取决于实现。

比如 拦截 超过100K 的消息

新建一个 maven 项目 在 `pom.xml` 中添加如下依赖

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.flume</groupId>
        <artifactId>flume-ng-core</artifactId>
        <version>1.9.0</version>
    </dependency>
</dependencies>
```



然后 实现 Interceptor 接口

```java
public class MyInterceptor implements Interceptor {
  private static final int MAX_MESSAGE_SIZE = 100 * 1024; // 100KB
 
  @Override
  public void initialize() {
    // 初始化拦截器
  }
 
  @Override
  public Event intercept(Event event) {
    // 拦截事件，并处理
    if (event.getBody().length > MAX_MESSAGE_SIZE) {
      // 如果日志消息过大，则丢弃该事件
      return null;
    }
    return event;
  }
 
  @Override
  public List<Event> intercept(List<Event> events) {
    // 拦截一组事件，并处理
    List<Event> intercepted = new ArrayList<>();
    for (Event event : events) {
      Event interceptedEvent = intercept(event);
      if (interceptedEvent != null) {
        intercepted.add(interceptedEvent);
      }
    }
    return intercepted;
  }
 
  @Override
  public void close() {
    // 关闭拦截器
  }
 
  public static class Builder implements Interceptor.Builder {
    @Override
    public Interceptor build() {
      return new MyInterceptor();
    }
 
    @Override
    public void configure(Context context) {
      // 从配置文件读取配置参数
    }
  }
}
```

将 项目打成 jar 包以后 上传到 flume 的 lib 目录下

如果想使用自定义的 拦截器 ，只需要在配置中设置即可

```properties
# 设置拦截器
a1.sources.r1.interceptors = i1
a1.sources.r1.interceptors.i1.type = com.example.MyInterceptor$Builder
```





