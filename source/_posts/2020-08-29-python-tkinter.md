---
layout: article
title: "[Python] Tkinter"
date: 2020-08-29 13:43
tags:
  - Python
  - Tkinter
---

tkinter 是 python 內建的 GUI。

<!--more-->

# Hello World

```python
from tkinter import *
from tkinter.ttk import *


class Application:
    def __init__(self):
        window = Tk()
        window.title('Hello World')
        window.geometry('800x600')
        window.mainloop()


if __name__ == "__main__":
    Application()
```

![](/assets/tkinter_hello_world.png)

# 基本元件介紹

## Widget

winfo_exists() 可檢查是否存在/顯示
winfo_children() 回傳底下所有 widget

## Frame

```python
Style().configure('red.TFrame', background='red')  # 自訂風格，字尾必為.TFrame，調整背景為紅色
f = Frame(window, style='user.TFrame')
```

清空 Frame

```python
def clear_frame(frame):
  """清空Frame"""
  for widgets in frame.winfo_children():
    widgets.destroy()

```

## Label

可透過['text']或 config 修改文字，config 可修改 UI 屬性

```python
label = Label(window, text='Label')
label['text'] = 'Hello'
label.config(text='World')
label.pack()
```

## Button

點擊事件有兩種設定方式：

1. 透過 command，只能處理左鍵單擊
2. 透過 bind，event.widget 可取得原物件

```python
btn = Button(window, text='Button', command=lambda: messagebox.showinfo(message='I am info'))
btn.bind('<Button-3>', lambda event: messagebox.showerror(message='I am error'))
btn.pack()
```

## Entry

修改文字有兩種設定方式

1. 透過 textvarialbe
2. 透過 insert，END 為 tkinter 常數，表示結尾

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

沒有 textvariable 屬性，只能用 insert()修改文字

```python
result_text = ScrolledText(master)
# 取代
result_text.delete(1.0, END)
result_text.insert(1.0, 'Hello World')

data = result_text.get(1.0, END)[:-1]  # 取值時，要小心結尾換行
selection = result_text.selection_get()  # 取得選取文字
lines = selection.split('\n')
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

def reset():
    """清除Treeview內容"""
    tv.delete(*tv.get_children())  # *一定要有，不然會出現 _tkinter.TclError: Cannot delete root item


vbar = Scrollbar(window, orient='vertical')
vbar.pack(side=RIGHT, fill=Y)
hbar = Scrollbar(window, orient='horizontal')
hbar.pack(side=BOTTOM, fill=X)

Style().configure('user.Treeview', rowheight=110)  # 自訂風格，字尾必為.Treeview，調整欄高為110
# height表顯示5行資料列，不含標題
tv = Treeview(window, xscrollcommand=hbar.set, yscrollcommand=vbar.set, show='headings', height=5, style='user.Treeview')
tv.pack(side=LEFT, fill=BOTH, expand=True)
tv['columns'] = 'id', 'name', 'password', 'auth_code'  # 不可以有中括弧
tv.heading('#1', text='id', command=lambda: sort_column('#1', False))
tv.heading('#2', text='name')
tv.heading('#3', text='password')
tv.heading('#4', text='auth_code')
tv.column('#1', width=130, stretch=NO)  # 調整欄位寬度
vbar.config(command=tv.yview)
hbar.config(command=tv.xview)
```

展開/收合 TreeView

```python
def tree_open(open):
    for child in tv.get_children():
        tv.item(child, open=open)
```

欄位可選取

```python
tv.bind('<Double-1>', _on_double_click)

def _on_double_click(event):
    col = tv.identify_column(event.x)
    rowid = tv.identify_row(event.y)
    box = tv.bbox(rowid, col)
    text = tv.item(rowid, 'values')[int(col[1:])-1]

    edit_entry = Entry(tv, width=box[2])
    edit_entry.insert(0, text)
    edit_entry.select_range(0, END)
    edit_entry.focus_set()
    edit_entry.bind('<FocusOut>', lambda event: event.widget.destroy())
    edit_entry.place(x=box[0], y=box[1], w=box[2], h=box[3])
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

## Toplevel

使用彈窗顯示 QR code

![](/assets/qrcode.png)

```python
import qrcode
from PIL import ImageTk

img = qrcode.make('Hello World')
popup = Toplevel()
popup.title('Hello World')
global photo  # 圖片需放全域保存
photo = ImageTk.PhotoImage(img.resize((200, 200)))
label = Label(popup, image=photo)
label.pack()
```

## 自訂元件

切記底下元素，例如\_label 的 master 要是 TextField 的 self，而不是 master。

```python
class TextField(Frame):
    def __init__(self, master, label, text, **kw):
        Widget.__init__(self, master, "ttk::frame", kw)
        _label = Label(self, text=label)
        _label.pack(side=LEFT)
        entry = Entry(self, width=len(text))
        entry.insert(END, text)
        entry.pack(side=RIGHT)

class Application:
    def __init__(self):
        window = Tk()
        window.title('Hello World')
        window.geometry('200x100')
        TextField(window, 'Name:', 'Mike').pack()
        window.mainloop()
```

![](/assets/custom_frame.png)

# 佈局

## Pack

```python
label.pack(anchor=W)  # 靠左呈現
```

## Grid

```python
label.grid(row=0, column=0, sticky=W)  # 靠左呈現
```

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
- &lt;Button-2&gt; windows 和 unix 是中鍵點擊，mac 是右鍵點擊，可用`platform.system() == 'Darwin'`判斷，Darwin 為 mac
- &lt;Button-3&gt; 右鍵點擊
- &lt;Double-1&gt; 左鍵連擊

參考<http://tcl.tk/man/tcl8.7/TkCmd/bind.html>

# lambda 與閉包

閉包會把上一層的變數偷渡捕捉進來自己的 function scope，Javascript 便是用閉包來模擬 class。

參考<https://medium.com/citycoddee/python進階技巧-4-lambda-function-與-closure-之謎-7a385a35e1d8>

舉例來說，當用 for 迴圈實作多個元件與按鈕，下面的寫法 submit()永遠只會取到最後被指派的元件。

```python
from tkinter import *
from tkinter.ttk import *
from tkinter.ttk import Button, Entry


def submit(value):
    print(value)

window = Tk()
for i in range(5):
    input = Entry(window)
    input.insert(0, str(i))
    input.grid(row=i, column=0)
    test = Button(window, text="Click me!", command=lambda: submit(input.get()))
    test.grid(row=i, column=1)

window.mainloop()
```

解法一，調整 lambda 寫法。

```python
test = Button(window, text="Click me!", command=lambda e=input.get(): submit(e))
```

解法二，將實作元件部分抽成 function，使用閉包捕捉上一層的變數來用。

```python
from tkinter import *
from tkinter.ttk import *
from tkinter.ttk import Button, Entry


def init_view(index):
    # 使用閉包抓住變數
    def submit():
        print(input.get())
    input = Entry(window)
    input.insert(0, str(index))
    input.grid(row=i, column=0)
    test = Button(window, text="Click me!", command=submit)
    test.grid(row=i, column=1)

window = Tk()
for i in range(5):
    init_view(i)

window.mainloop()
```
