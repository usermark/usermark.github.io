---
layout: article
title: '[Python] SQLite應用'
date: 2023-02-17 07:24
tags: Python SQLite
---
紀錄開發上常遇到的問題，避免重複踩坑。
<!--more-->
# 建立Table

若db檔案不存在，會自動建立。
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

注意：要小心別用到關鍵字作為表格名稱，例如order, group，參考 <https://www.sqlite.org/lang_keywords.html>

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

# 取得DB底下的所有Table

參考<https://stackoverflow.com/questions/305378/list-of-tables-db-schema-dump-etc-using-the-python-sqlite3-api>
```python
conn = sqlite3.connect('database.db')
curs = conn.cursor()
curs.execute("SELECT name FROM sqlite_master WHERE type='table';")
print(curs.fetchall())
```

# 轉換Table資料為dict

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