---
layout: article
title: '[Python] 應用'
date: 2022-04-02 16:45
tags: Python
---
紀錄開發上常遇到的問題，避免重複踩坑。
<!--more-->
# Requests紀錄API請求和回應

參考<https://stackoverflow.com/questions/16337511/log-all-requests-from-the-python-requests-module>
```python
import logging
from http.client import HTTPConnection

def debug_requests_on():
    '''Switches on logging of the requests module.'''
    HTTPConnection.debuglevel = 1

    logging.basicConfig()
    logging.getLogger().setLevel(logging.DEBUG)
    requests_log = logging.getLogger("requests.packages.urllib3")
    requests_log.setLevel(logging.DEBUG)
    requests_log.propagate = True

def debug_requests_off():
    '''Switches off logging of the requests module, might be some side-effects'''
    HTTPConnection.debuglevel = 0

    root_logger = logging.getLogger()
    root_logger.setLevel(logging.WARNING)
    root_logger.handlers = []
    requests_log = logging.getLogger("requests.packages.urllib3")
    requests_log.setLevel(logging.WARNING)
    requests_log.propagate = False
```

# requests_html出現SSLCertVerificationError

第一次執行會有SSLCertVerificationError問題
```shell
[W:pyppeteer.chromium_downloader] start chromium download.
Download may take a few minutes.
Traceback (most recent call last):
  略...
ssl.SSLCertVerificationError: [SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: unable to get local issuer certificate (_ssl.c:1056)
```
修正如下，參考<https://github.com/miyakogi/pyppeteer/issues/219>
```shell
pip install -U "urllib3<1.25"
```

# 輸出yaml換行格式

```python
def repr_str(dumper: yaml.SafeDumper, data):
    if '\n' in data:
        return dumper.represent_scalar('tag:yaml.org,2002:str', data, style='|')
    return dumper.represent_str(data)

yaml.add_representer(str, repr_str, Dumper=yaml.SafeDumper)
print(yaml.safe_dump(foo))
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

# os.popen()在Windows下出現 UnicodeDecodeError: 'cp950' codec can't decode byte

os.popen()是封裝subprocess.Popen()，並用io.TextIOWrapper轉換bytes輸出成str的結果，而Windows下的cmd指令預設為cp950，會造成解析錯誤。

```python
proc = subprocess.Popen(cmd,
                        shell=True,
                        stdout=subprocess.PIPE,
                        bufsize=buffering)
return _wrap_close(io.TextIOWrapper(proc.stdout), proc)
```

所以針對Windows自行實作popen()，調整如下

```python
def popen(command):
    if platform.system() == 'Darwin':
        return os.popen(command)
    proc = subprocess.Popen(command, stdout=subprocess.PIPE)
    return io.TextIOWrapper(proc.stdout, encoding='UTF-8')
```

# 取得DB底下的所有Table

參考<https://stackoverflow.com/questions/305378/list-of-tables-db-schema-dump-etc-using-the-python-sqlite3-api>
```python
conn = sqlite3.connect('database.db')
curs = conn.cursor()
curs.execute("SELECT name FROM sqlite_master WHERE type='table';")
print(curs.fetchall())
```

# Flask保留json排序

參考<https://stackoverflow.com/questions/54446080/how-to-keep-order-of-sorted-dictionary-passed-to-jsonify-function>
```python
app = Flask(__name__)
app.config['JSON_SORT_KEYS'] = False
```