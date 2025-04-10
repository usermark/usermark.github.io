---
layout: article
title: "[Python] SQL應用"
date: 2023-02-17 07:24
tags:
  - Python
  - SQL
  - SQLite
  - DB2
  - MSSQL
---

紀錄開發上常遇到的問題，避免重複踩坑。

<!--more-->

# 連線資料庫

_SQLite_

```python
import sqlite3

conn = sqlite3.connect('enterprise.db')
```

_DB2_
需安裝 ibm-db 套件

```sh
pipenv install ibm-db
```

參考 https://github.com/ibmdb/python-ibmdb?tab=readme-ov-file#installation
將 C:\Users\{使用者}\AppData\Local\Programs\Python\Python{版本號}\Lib\site-packages 底下 clidriver 資料夾移動到 C:\Program Files

```python
import os
os.add_dll_directory('C:\\Program Files\\clidriver\\bin')
import ibm_db

conn = ibm_db.connect('DATABASE=enterprise;HOSTNAME=ip;PORT=50000;PROTOCOL=TCPIP;UID=user;PWD=pwd', '', '')
```

_MSSQL_

```python
import pymssql

conn = pymssql.connect(host="ip:1433",
                       user="user",
                       password="pwd",
                       database="enterprise",
                       charset="utf8")
```

# 建立 Table

若 db 檔案不存在，會自動建立。

```python
conn = sqlite3.connect('enterprise.db')
curs = conn.cursor()
curs.execute('''CREATE TABLE IF NOT EXISTS zoo
                (critter VARCHAR(20) PRIMARY KEY,
                 count INT,
                 damages FLOAT)''')
curs.close()
conn.close()
```

注意：要小心別用到關鍵字作為表格名稱，例如 order, group，參考 <https://www.sqlite.org/lang_keywords.html>

# 插入資料

```python
conn = sqlite3.connect('enterprise.db')
curs = conn.cursor()
curs.execute('INSERT INTO zoo VALUES (?, ?, ?)', ('duck', 5, 0.0))
curs.execute("INSERT INTO zoo VALUES ('{0}', {1}, {2})".format('bear', 2, 1000.0))  # 第二種寫法
curs.close()
conn.commit()  # 必要
conn.close()
```

# 查詢資料

_SQLite_

```python
conn = sqlite3.connect('enterprise.db')
curs = conn.cursor()
curs.execute('SELECT critter FROM zoo WHERE damages BETWEEN 0 AND 1000')
print(curs.fetchall())
curs.close()
conn.close()
```

輸出結果

```
[('duck',), ('bear',)]
```

_DB2_

```python
conn = ibm_db.connect('DATABASE=enterprise;HOSTNAME=ip;PORT=50000;PROTOCOL=TCPIP;UID=user;PWD=pwd', '', '')
stmt = ibm_db.exec_immediate(conn, 'SELECT critter FROM zoo WHERE damages BETWEEN 0 AND 1000')
result = ibm_db.fetch_assoc(stmt)  # 只有第一筆
while result != False:
    for index, item in enumerate(result.keys()):
        print(item)
    result = ibm_db.fetch_assoc(stmt)  # 取下一筆
```

## 取得 DB 底下的所有 Table

_SQLite_
參考<https://stackoverflow.com/questions/305378/list-of-tables-db-schema-dump-etc-using-the-python-sqlite3-api>

```python
conn = sqlite3.connect('database.db')
curs = conn.cursor()
curs.execute("SELECT name FROM sqlite_master WHERE type='table';")
print(curs.fetchall())
```

# 轉換 Table 資料為 dict

```python
def dict_factory(cursor, row):
    """轉換tuple為dict"""
    return dict((col[0], row[idx]) for idx, col in enumerate(cursor.description))

conn = sqlite3.connect('database.db')
curs = conn.cursor()
curs.execute("SELECT * FROM Product")
result = [dict_factory(curs, item) for item in curs.fetchall()]
curs.close()
conn.close()
```
