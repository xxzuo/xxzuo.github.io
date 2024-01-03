---
title: hexo常用命令
author: xxzuo
tags:
  - hexo
categories:
  - 环境配置
date: 2023-11-24 08:53:45
---

摘录自 [官方文档](https://hexo.io/zh-cn/docs/commands.html)

## 新建文章

新建一篇文章。如果没有设置 `layout` 的话，默认使用 [_config.yml](https://hexo.io/zh-cn/docs/configuration) 中的 `default_layout` 参数代替。如果标题包含空格的话，请使用引号括起来。

```bash
$ hexo new [layout] <title>
```

参数

|参数|描述|
|---|---|
|`-p`, `--path`|自定义新文章的路径|
|`-r`, `--replace`|如果存在同名文章，将其替换|
|`-s`, `--slug`|文章的 Slug，作为新文章的文件名和发布后的 URL|


e.g
```bash
$ hexo new "post title with whitespace"
```


## 生成静态文件
```bash
$ hexo generate
```

生成静态文件。

|选项|描述|
|---|---|
|`-d`, `--deploy`|文件生成后立即部署网站|
|`-w`, `--watch`|监视文件变动|
|`-b`, `--bail`|生成过程中如果发生任何未处理的异常则抛出异常|
|`-f`, `--force`|强制重新生成文件  <br>Hexo 引入了差分机制，如果 `public` 目录存在，那么 `hexo g` 只会重新生成改动的文件。  <br>使用该参数的效果接近 `hexo clean && hexo generate`|
|`-c`, `--concurrency`|最大同时生成文件的数量，默认无限制|

该命令可以简写为
```bash
$ hexo g
```

## 部署网站

```bash
$ hexo deploy
```

部署网站。

|参数|描述|
|---|---|
|`-g`, `--generate`|部署之前预先生成静态文件|

该命令可以简写为：
```bash
$ hexo d
```

一般流程为

```bash
# 新建文章
hexo new "new doc"

# 编辑刚创建的文章

# 生成静态文件
hexo g

# 部署到 github
hexo d

```


