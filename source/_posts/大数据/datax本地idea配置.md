---
categories:
  - 大数据
---
1. 下载源码打包
[alibaba/DataX: DataX是阿里云DataWorks数据集成的开源版本。 (github.com)](https://github.com/alibaba/DataX)

执行打包命令
```shell
mvn -U clean package assembly:assembly -Dmaven.test.skip=true
```

2. 配置启动参数
在 idea 中找到  `com.alibaba.datax.core.Engine` 类

在 `vm option` 中写上 打包后的 datax 目录
> -Ddatax.home=/xxx/DataX/target/datax/datax


`Program arguments` 中加上 要执行的 json 文件
> -mode  standalone  -jobid  -1  -job  /xxx/job.json