---
layout: post
title: '[Python] tkinter'
date: 2020-08-29 13:43
categories: 
---
tkinter是python內建的GUI。

# Hello World
```python
from tkinter import *


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
entry.insert(END, ' World')
entry.pack()
```
## Combobox
下拉式選單
```python
cb = Combobox(window)
cb['values'] = list(range(5))
cb.config(state='readonly')  # 可設成唯讀
cb.current(0)  # 指定選取索引0
cb.pack()
```

# Bind
- &lt;Button-1&gt; 左鍵點擊
- &lt;Button-2&gt; windows和unix是中鍵點擊，mac是右鍵點擊，可用`platform.system() == 'Darwin'`判斷
- &lt;Button-3&gt; 右鍵點擊
- &lt;Double-1&gt; 左鍵連擊

參考連結：[http://tcl.tk/man/tcl8.7/TkCmd/bind.htm](http://tcl.tk/man/tcl8.7/TkCmd/bind.htm)