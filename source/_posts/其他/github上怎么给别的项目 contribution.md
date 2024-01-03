---
title: github上怎么给别的项目 contribution
author: xxzuo
categories:
  - 其他
date: 2023-12-26 16:32:03
tags:
---

1.准备仓库
首先 fork 你想贡献的 仓库 这里以 doris 为例


克隆到本地
```
git clone https://github.com/(your_github_name)/doris.git
```

添加远程仓库
```
git remote add upstream https://github.com/apache/doris.git 
git remote -v
```

2.创建分支
- 切换到 fork 的 master 分支，拉取最新代码，创建本次的分支。
```
git checkout master  
git fetch upstream  
git rebase upstream/master  
git push origin master # 可选操作  
git checkout -b issueNo
```


3.提交 PR
开发完代码以后 ，将该分支 push 到自己fork 的git仓库
然后提交一个 PR 

