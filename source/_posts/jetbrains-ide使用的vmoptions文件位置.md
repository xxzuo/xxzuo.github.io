---
title: jetbrains-ide使用的vmoptions文件位置
author: xxzuo
date: 2022-12-02 14:53:26
tags:
- jetbrains
categories:
---

# IDE使用了哪个vmoptions文件? 

IDE依次检查下面各项,若满足,则使用相应的文件

## 1.环境变量指向的vmoptions文件

如果`<IDE>_VM_OPTIONS`环境变量存在且指向的vmoptions文件也存在, 则使用该文件.其中<IDE>是jetbrains IDE的代号,比如 IDEA / PYCHARM 等

环境变量里找到 以 _VM_OPTIONS 结尾的环境变量, 如下, 对应的值就是相应IDE使用的vmoptions文件了

![](https://raw.githubusercontent.com/xxzuo/pic_host/main/image2022-12-1_11-12-53.png)



## 2.配置目录

### 2.1 如果当前IDE是Toolbox安装的,则使用IDE安装目录下的 <version>.vmoptions 文件,其中<version>是IDE的版本号



### 2.2 如果不是通过toolbox安装,而是独立安装的IDE, 则使用配置目录下的文件

**常见的配置目录**

| 操作系统                                                    | 配置目录                                                   |
| :---------------------------------------------------------- | :--------------------------------------------------------- |
| 操作系统                                                    | 配置目录                                                   |
| Windows                                                     | %APPDATA%\JetBrains\<product><version>                     |
| 示例:                                                       |                                                            |
| C:\Users\JohnS\AppData\Roaming\JetBrains\IntelliJIdea2022.1 |                                                            |
| MacOS                                                       | ~/Library/Application Support/JetBrains/<product><version> |
| 示例:                                                       |                                                            |
| ~/Library/Application Support/JetBrains/IntelliJIdea2022.1  |                                                            |
| Linux                                                       | ~/.config/JetBrains/<product><version>                     |
| 示例:                                                       |                                                            |
| ~/.config/JetBrains/IntelliJIdea2022.1                      |                                                            |



## 3. 默认位置下 [即bin目录]

如果以上位置都没有vmoptions文件, 则使用默认位置下 [即bin目录]的文件

以 idea为例

| 操作系统 | 默认位置                                      |
| :------- | :-------------------------------------------- |
| Windows  | <IDE_HOME>\bin\idea64.exe.vmoptions           |
| macOS    | IntelliJ IDEA.app/Contents/bin/idea.vmoptions |
| Linux    | <IDE_HOME>/bin/idea64.vmoptions               |
