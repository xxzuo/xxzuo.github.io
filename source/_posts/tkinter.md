---
title: tkinter
author: xxzuo
date: 2022-08-03 23:54:08
tags:
- tkinter
- python-gui
categories:
- python
---



由于会用到 `python`写一些小工具，使用 `tkinter`画一些简单的界面

在此记录一下 `tkinter` 的一些用法

一个简单的 `tkinter`程序至少应包含以下四个部分：

- `import  tkinter`
- 创建窗口
- 添加控件，以及相应的事件函数
- 通过`mainloop`来显示主窗口

```python
# 导入 tk 包
import tkinter as tk
# 调用Tk()创建主窗口
root =tk.Tk()
#开启主循环，让窗口处于显示状态
root.mainloop()
```



# tkinter窗口

创建窗口之后

可以自定义窗口属性

常见的属性有下面几个:

我们可以定义 窗口的 标题 Title

> root.title()

也可以定义窗口的 大小(长 * 宽) 以及 窗口的位置 （x 坐标 y 坐标）

> root.geometry("400x200+900+200")

或者 允许`tkinter`根窗口根据用户需要更改其大小

> root.resizable(height = None, width = None)

窗口的属性如下

| 函数                     | 说明                                                         |
| ------------------------ | ------------------------------------------------------------ |
| window.title("my title") | 接受一个字符串参数，为窗口起一个标题                         |
| window.resizable()       | 否允许用户拉伸主窗口大小，默认为可更改，当设置为 resizable(0,0)或者resizable(False,False)时不可更改 |
| window.geometry()        | 设定主窗口的大小以及位置，当参数值为 None 时表示获取窗口的大小和位置信息。 |
| window.quit()            | 关闭当前窗口                                                 |
| window.update()          | 刷新当前窗口                                                 |



















