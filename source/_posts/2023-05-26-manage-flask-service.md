---
layout: article
title: "[Python] Windows下管理Flask服務"
date: 2023-05-26 23:21
tags:
  - Python
  - Flask
---

因習慣透過 pipenv 指立建立 Flask，例如

```sh
pipenv run python mock_server.py
```

要正確地執行，花費些時間研究，因此特別記錄下來。

<!--more-->

# 關閉所有 Python 服務

```sh
@echo off
setlocal EnableDelayedExpansion

for /f "tokens=2" %%a in ('tasklist /fi "imagename eq python.exe" /nh') do (
    set pid=%%~a
    set "cmd="
    for /f "tokens=2,*" %%b in ('wmic process where "ProcessId=!pid!" get commandline /format:list ^| find "="') do (
        set cmd=%%~b
    )
    echo !pid! !cmd!
    taskkill /f /pid !pid!
)
```

# 使用 nssm 註冊服務

首先，要知道 pipenv 產生的虛擬環境在哪

```sh
pipenv --venv
```

假設輸出結果如下

```sh
C:\Users\mike\.virtualenvs\mock_server-ASS21Y9K
```

參考<https://www.mssqltips.com/sqlservertip/7325/how-to-run-a-python-script-windows-service-nssm/>

先下載[nssm](https://nssm.cc/download)，解壓縮後，需用系統管理員權限開啟 cmd，並切換到目錄底下

```sh
cd C:\nssm-2.24\win64
```

接著執行 nssm 來建立服務

```sh
nssm install "MockServer" "C:\Users\mike\.virtualenvs\mock_server-ASS21Y9K\Scripts\python.exe" "mock_server.py"
nssm set "MockServer" AppDirectory "C:\nginx\html\mock_server"
nssm set "MockServer" AppStdout "C:\nginx\html\mock_server\out_file.txt"
nssm set "MockServer" AppStderr "C:\nginx\html\mock_server\error_file.txt"
```

啟動或停止服務

```sh
nssm start "MockServer"
nssm stop "MockServer"
```

# 透過 API 關閉 Flask 服務

試過上述兩個方法，還是這個方法最簡單乾淨，只要留意好權限控管，避免被惡意呼叫關閉即可。

```python
@app.route('/shutdown', methods=['POST'])
def shutdown():
    os._exit(0)
```
