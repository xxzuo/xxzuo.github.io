---
title: hexo-auto-category配置
author: xxzuo
tags:
  - hexo插件
categories:
  - 环境配置
date: 2023-07-14 00:20:36
---

以往在写 blog 的时候 ，通常是使用 
```
new page "xxx"
```
来新建文章，但是这样有一个问题 就是新建的文件全部都堆在 `/_post` 文件下，blog 多了以后非常难维护，如果我们 手动做了 分类，这个时候因为 我们 在 front-matter 中 categories 是手动维护的，，如果文件夹出现了修改或者说 blog 的移动，这个时候还需要手动的去修改 categories 非常麻烦。
这个时候 我们可以用 [hexo-auto-category](https://github.com/xu-song/hexo-auto-category)) 这个插件 来帮助我们自动分类。这个插件 可以根据blog 所处的文件夹 来自动生成 categories, 省去了手动维护的烦恼.

安装见 [【Hexo插件系列】日志的自动分类插件 hexo-auto-category | ESON](https://blog.eson.org/pub/e2f6e239/)

#### 安装
```
npm install hexo-auto-category --save
```

#### 配置
在 `_config.yml` 中添加
```
# Generate categories from directory-tree  
# Dependencies: https://github.com/xu-song/hexo-auto-category  
# depth: the depth of directory-tree you want to generate, should > 0  
auto_category:  
 enable: true  
 depth: 1
```

`depth : 1` 表示只想生成第一级目录


#### 编译&部署
```
hexo clean && hexo g && hexo d
```








