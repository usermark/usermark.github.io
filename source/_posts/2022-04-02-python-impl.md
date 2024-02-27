---
layout: article
title: '[Python] 應用'
date: 2022-04-02 16:45
tags:
- Python
- AES
---
紀錄開發上常遇到的問題，避免重複踩坑。
<!--more-->
# 讀取yaml檔

需安裝pyyaml套件
```sh
pipenv install pyyaml
```

```python
import yaml

with open('config.yml', encoding='UTF-8') as stream:
    configs = yaml.safe_load(stream)
```

# 讀取zip檔內的文字檔內容

```python
import zipfile

with zipfile.ZipFile('path/to/zip_file', 'r') as zip_ref:
    linesb = zip_ref.open(zip_ref.namelist()[0]).readlines()  # 開啟zip檔內第一個檔案
    print(''.join([lineb.decode('UTF-8') for lineb in linesb]))
```

# 解壓縮rar檔

需安裝unrar套件，並新增環境變數 UNRAR_LIB_PATH = C:\Program Files (x86)\UnrarDLL\x64\UnRAR64.dll
```sh
pipenv install unrar
```

```python
from unrar import rarfile

with rarfile.RarFile('path/to/rar_file') as rar_ref:
    rar_ref.extract(rar_ref.namelist()[0], 'unrar/to/path')  # 解壓縮第一個檔案
```

# 開啟資料夾

```python
import subprocess

def open_folder(path):
    """開啟資料夾"""
    if platform.system() == 'Darwin':
        subprocess.call(['open', path])
    else:
        subprocess.call(['start', path], shell=True)
```

# NAS連線並取檔

需安裝pysmb套件
```sh
pipenv install pysmb
```

```python
from smb.SMBConnection import SMBConnection

conn = SMBConnection('user', 'pwd', '', '', domain='your_domain')
if not conn.connect('ip', port=445, timeout=10):
    print('連線失敗')
    return
with open('path/to/output_file', 'wb') as output_file:
    conn.retrieveFile('share_folder', 'path/to/file', output_file)
conn.close()
```

## 支援SMB2/3

需安裝smbprotocol套件
```sh
pipenv install smbprotocol
```

參考 https://github.com/jborean93/smbprotocol/blob/master/examples/high-level/file-management.py
```python
from smbclient import register_session, open_file

register_session('ip', username='user', password='pwd')

with open('path/to/output_file', 'wb') as output_file:
    with open_file(r"\\ip\path\to\file", mode="rb") as fd:
        output_file.write(fd.read())
```

# AES加解密

需安裝pycryptodome套件
```sh
pipenv install pycryptodome
```

通常拿到的key是用Hex表示的字串，要用bytes.fromhex(key)轉換成bytes。

加密後的結果為bytes，無法直接編碼成字串，所以中間會先轉成Base64。因此解密時，資料已經過Base64編碼，需Base64解碼後才丟進AES解密。

```python
import base64

from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad


aes_key = 'C6DFE59F58353AEF73549370CD98F0A9547EF16C74974C16520F0C868ECB78EC'
# b'\xc6\xdf\xe5\x9fX5:\xefsT\x93p\xcd\x98\xf0\xa9T~\xf1lt\x97L\x16R\x0f\x0c\x86\x8e\xcbx\xec'
data = 'Hello world'

def aes_encrypt(key, text):
    """AES加密"""
    cipher = AES.new(bytes.fromhex(key), AES.MODE_ECB)
    result = cipher.encrypt(pad(text.encode(), AES.block_size))
    result_b64 = base64.b64encode(result)
    result_str = result_b64.decode()
    return result_str

def aes_decrypt(key, text):
    """AES解密"""
    text_b64 = text.encode()
    text_b = base64.b64decode(text_b64)
    cipher = AES.new(bytes.fromhex(key), AES.MODE_ECB)
    result = unpad(cipher.decrypt(text_b), AES.block_size)
    result_str = result.decode()
    return result_str

if __name__ == '__main__':
    text = aes_encrypt(aes_key, data)
    print(text)
    data = aes_decrypt(aes_key, text)
    print(data)
```

輸出結果
```
LE1WdHzcoXb7ul3S9b5otQ==
Hello world
```

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

# Flask保留json排序

參考<https://stackoverflow.com/questions/54446080/how-to-keep-order-of-sorted-dictionary-passed-to-jsonify-function>
```python
app = Flask(__name__)
app.config['JSON_SORT_KEYS'] = False
```