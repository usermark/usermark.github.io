---
title: "[Docker] 上架 Flask 網站到 Google Cloud Run"
date: 2026-01-30 16:46:48
tags:
  - Docker
  - Flask
  - Google Cloud Run
---
放了很久，遲遲未整理和紀錄，今天終於下定決心。
<!--more-->

# 使用 docker 運行 Flask 網站

寫一個簡單的 main.py
```python
import flask

app = flask.Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello World'

if __name__ == '__main__':
    app.run('0.0.0.0', 8000, debug=True)
```

產生 requirements.txt
```
flask
gunicorn
```

Dockerfile 內容如下
```
FROM python:3.11
WORKDIR /app
COPY . /app
RUN pip install --upgrade -r requirements.txt
CMD gunicorn main:app
```

參數說明：
* FROM 為運行的 image
* WORKDIR 為工作目錄
* COPY 複製專案目錄至指定目錄(/app)
* RUN 運行 image 時，要多跑的指令 
* CMD 每次啟動容器時所執行的指令

根據 Dockerfile 打包成 image
```shell
docker image build -t hello_world .
```

新建並啟動 container
```shell
docker run -p 8000:8000 -d --name hello01 hello_world
```

接著打開瀏覽器 http://localhost:8000

# 上傳到 Google Cloud Run 執行

# 進階：使用 Firestore Database

既然服務都放在 GCP，就來整合下 Firestore

```shell
pip install --upgrade firebase-admin
```

**參考資料**

1. [實作 Dockerfile + flask 教學](https://www.maxlist.xyz/2020/01/11/docker-flask/)