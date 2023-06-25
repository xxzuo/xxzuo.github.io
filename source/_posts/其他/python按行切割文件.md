---
title: python按行切割文件
tags:
  - python
categories:
  - 其他
date: 2022-06-20 23:03:47
---

# 利用python 按行分割文件

代码如下

```python
import os
import time
import tkinter as tk
from tkinter import filedialog
from tkinter.messagebox import showinfo, showwarning, showerror
 
 
def mkSubFile(lines, head, srcName, sub):
    [des_filename, extname] = os.path.splitext(srcName)
    filename = des_filename + '_' + str(sub) + extname
    print('make file: %s' % filename)
    fout = open(filename, 'w')
 
    try:
        fout.writelines([head])
        fout.writelines(lines)
        return sub + 1
    finally:
        fout.close()
 
 
def splitByLineCount(filename, count):
    fin = open(filename, 'r')
    try:
        head = fin.readline()
        buf = []
        sub = 1
        for line in fin:
            buf.append(line)
            if len(buf) == count:
                sub = mkSubFile(buf, head, filename, sub)
                buf = []
        if len(buf) != 0:
            sub = mkSubFile(buf, head, filename, sub)
    finally:
        fin.close()
 
 
def init():
    entryNum['state'] = "disable"
    btnConfirm['state'] = "normal"
 
 
def confirm():
    f_path = filedialog.askopenfilename()
    inputFilePath.set(f_path)
 
 
def clear():
    inputFilePath.set("")
    row.set("")
 
 
def startSplitFile():
    if len(inputFilePath.get()) == 0:
        showwarning(title="警告",
                    message="未选择文件路径!")
        return
    try:
        if int(row.get()) <= 0:
            showwarning(title="警告",
                        message="输入的不是正整数!")
            return
    except:
        showwarning(title="警告",
                    message="输入的不是整数!")
        return
    count = int(row.get())
    begin = time.time()
    splitByLineCount(inputFilePath.get(), count)
    end = time.time()
    print('time is %d seconds ' % (end - begin))
 
 
def closeWindow():
    root.destroy()
 
 
if __name__ == '__main__':
    root = tk.Tk()
    root.title("File Split")
    root.resizable(False, False)
    root.geometry("600x100+480+320")
 
    mess = tk.Label(root, text="请选择要切分的文件：")
    mess.place(x=20, y=10, width=200, height=20)
 
    inputFilePath = tk.StringVar(root, value='')
    entryNum = tk.Entry(root, width=80, textvariable=inputFilePath)
    entryNum.place(x=220, y=10, width=260, height=20)
 
    btnConfirm = tk.Button(root, text='选择文件', command=confirm)
    btnConfirm.place(x=500, y=10, width=70, height=20)
 
    mess1 = tk.Label(root, text="请输入切分的文件行数：")
    mess1.place(x=20, y=40, width=200, height=20)
 
    row = tk.StringVar(root, value='')
    entryNum1 = tk.Entry(root, width=80, textvariable=row)
    entryNum1.place(x=220, y=40, width=200, height=20)
 
    btnStart = tk.Button(root, text='清空', command=clear)
    btnStart.place(x=260, y=70, width=70, height=20)
 
    btnSet = tk.Button(root, text='开始切分', command=startSplitFile)
    btnSet.place(x=125, y=70, width=70, height=20)
 
    init()
    root.protocol("WM_DELETE_WINDOW", closeWindow)
    root.mainloop()
```

