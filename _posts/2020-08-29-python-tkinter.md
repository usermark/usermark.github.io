---
layout: article
title: '[Python] tkinter'
date: 2020-08-29 13:43
tags: Python Tkinter
---
tkinter是python內建的GUI。
<!--more-->
# Hello World
```python
from tkinter import *
from tkinter.ttk import *


class Application:
    def __init__(self):
        window = Tk()
        window.title('Hello World')
        window.mainloop()


if __name__ == "__main__":
    Application()
```

# 基本元件介紹
## Label
可透過['text']或config修改文字，config可修改UI屬性
```python
label = Label(window, text='Label')
label['text'] = 'Hello'
label.config(text='World')
label.pack()
```
## Button
點擊事件有兩種設定方式：
1. 透過command，只能處理左鍵單擊
2. 透過bind，event.widget可取得原物件

```python
btn = Button(window, text='Button', command=lambda: messagebox.showinfo(message='I am info'))
btn.bind('<Button-3>', lambda event: messagebox.showerror(message='I am error'))
btn.pack()
```
## Entry
修改文字有兩種設定方式
1. 透過textvarialbe
2. 透過insert，END為tkinter常數，表示結尾

```python
value = StringVar()
value.set('Hello')
entry = Entry(window, textvariable=value)
# 插入，結果Hello World
entry.insert(END, ' World')
# 取代，結果Test
entry.delete(0, END)
entry.insert(0, 'Test')
entry.pack()
```
## ScrolledText
沒有textvariable屬性，只能用insert()修改文字

## Combobox
下拉式選單
```python
cb = Combobox(window)
cb['values'] = list(range(5))
cb.config(state='readonly')  # 可設成唯讀
cb.current(0)  # 指定選取索引0
cb.pack()
```

## TreeView
可捲動的表格
```python
def sort_column(col, reverse):
    """依此欄位排序"""
    items = [(tv.set(child, col), child) for child in tv.get_children()]  #取出欄位值對應項目
    items.sort(reverse=reverse)

    for index, (val, child) in enumerate(items):
        tv.move(child, '', index)  # 依新的索引移動項目

    tv.heading(col, command=lambda: sort_column(col, not reverse))  # 重新定義方法為反向排序

vbar = Scrollbar(window)
vbar.pack(side=RIGHT, fill=Y)

Style().configure('user.Treeview', rowheight=110)  # 自訂風格，調整欄高為110
tv = Treeview(window, yscrollcommand=vbar.set, show='headings', style='user.Treeview')
tv.pack(side=LEFT, fill=BOTH, expand=True)
tv['columns'] = 'id', 'name', 'password', 'auth_code'
tv.heading('#1', text='id', command=lambda: sort_column('#1', False))
tv.heading('#2', text='name')
tv.heading('#3', text='password')
tv.heading('#4', text='auth_code')
vbar.config(command=tv.yview)
```

## PanedWindow
可調整的佈局元件

```python
pw = PanedWindow(orient=HORIZONTAL)
sidebar = Frame(pw, width=200, height=200, relief=GROOVE)
main = Frame(pw, width=400, height=400, relief=GROOVE)

pw.pack(fill=BOTH, expand=True)

pw.add(sidebar)
pw.add(main)
```

## Widget
winfo_exists() 可檢查是否存在/顯示

winfo_children() 回傳底下所有widget

## Grid
自動佈局，每三個項目為一列
```python
def auto_grid(parent, widget):
    index = len(parent.winfo_children()) - 1
    widget.config(width=10)
    # print(f'index={index}, row={index // 3}, column={index % 3}')
    widget.grid(row=index // 3, column=index % 3, ipady=12)
```

# Bind
- &lt;Button-1&gt; 左鍵點擊
- &lt;Button-2&gt; windows和unix是中鍵點擊，mac是右鍵點擊，可用`platform.system() == 'Darwin'`判斷，Darwin為mac
- &lt;Button-3&gt; 右鍵點擊
- &lt;Double-1&gt; 左鍵連擊

參考<http://tcl.tk/man/tcl8.7/TkCmd/bind.html>