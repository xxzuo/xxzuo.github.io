---
title: hexo部署问题
date: 2021-04-29 00:56:27
tags:
- hexo
categories: 
- 配置相关
---

### 使用hexo d 命令部署之后 访问页面不显示

#### 原因

因为之前看的网上教程，在`_config.yml`文件中设置的

```yml
deploy:
  type: git
  repository: 
  branch: master
```

中将branch 设置为了 master,这是因为github现在新建仓库的主分支为main [github/renaming: Guidance for changing the default branch name for GitHub repositories](https://github.com/github/renaming)

所以现在github pages 调用的分支为maIn分支



#### 解决方法

打开博客仓库的settings

<img src="/img/hexo配置/hexo1.jpg">

拉到最下面看到github pages配置

<img src="/img/hexo配置/hexo2.jpg">

然后将分支改为master即可

<img src="/img/hexo配置/hexo3.jpg">