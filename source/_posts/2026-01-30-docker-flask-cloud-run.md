---
title: "[Docker] 上架 Flask 網站到 Google Cloud Run"
date: 2026-01-30 16:46:48
tags:
  - Docker
  - Flask
  - Google Cloud Run
  - Firestore
---
放了很久，遲遲未整理和紀錄，今天終於下定決心。
<!--more-->

# 使用 docker 運行 Flask 網站

建立一個 hello_world 資料夾，寫一個簡單的 main.py
```python
import flask

app = flask.Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello World'

if __name__ == '__main__':
    app.run('0.0.0.0', 8000, debug=True)
```

建立 requirements.txt
```
flask
```

建立 Dockerfile
```
FROM python:3.11
WORKDIR /app
COPY . /app
RUN pip install --upgrade -r requirements.txt
CMD python main.py
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
看到 Hello World 便表示成功囉！

# 上傳到 Google Cloud Run 執行

參考 [\[GCP\] GCP上傳映像檔至 Artifact Registry](/2024/03/21/upload-gcp-artifact-Registry/) 這篇的作法

輸入指令下 tag
```shell
docker tag hello_world asia-east1-docker.pkg.dev/xxx/hub/hello_world:v1
```

輸入指令上傳 image
```shell
docker push asia-east1-docker.pkg.dev/xxx/hub/hello_world:v1
```

進入GCP頁面 > 至「Cloud Run」選擇「部署容器」
![](/assets/cloud_run.png)

選擇剛才上傳至 Artifact Registry 的 image，注意底下的 port 預設是 8080，要改成 8000 (或指定 port)。
![](/assets/cloud_run2.png)

待部署完成後，就可以打開GCP提供的網址做確認。
![](/assets/cloud_run3.png)

下次還有修改 image 的話，可直接用下列指令建立
```shell
docker image build -t asia-east1-docker.pkg.dev/xxx/hub/hello_world:v2 .
```

# 進階：使用 Firestore Database

既然服務都放在 GCP，就來整合下 Firestore

新增 Firebase Admin SDK
```shell
pip install --upgrade firebase-admin
```

先在 Firebase 建立好專案，再到 GCP 控制台中，前往「IAM 與管理」>「服務帳戶」，找到「firebase-adminsdk」開頭的帳號，進入並建立金鑰，會取得 JSON 檔案。複製一份到專案資料夾底下，並命名為 firebase-adminsdk.json

修改 main.py
```python
from datetime import datetime, timedelta, timezone
from flask import request

from firebase_admin import initialize_app, firestore, credentials
import flask

cred = credentials.Certificate("firebase-adminsdk.json")
initialize_app(cred)
db = firestore.client()

app = flask.Flask(__name__)

@app.route('/')
def index():
    docs = db.collection("todo").stream()
    result = ''
    for doc in docs:
        result += f'{doc.id} => {doc.to_dict()["content"]}<br>'
    return result

@app.post('/todo')
def todo():
    payload = request.get_json()
    title = payload['title']
    expires_at = datetime.now(timezone.utc) + timedelta(minutes=30)
    db.collection("todo").document(title).set({
        'content': payload['content'],
        'expires_at': expires_at
    })
    return 'OK'

if __name__ == '__main__':
    app.run('0.0.0.0', 8000, debug=True)
```

大致說明下功能，為簡單的待辦清單，GET 取得所有清單，POST 用來新增一筆事項。

修改 requirements.txt
```
flask
firebase-admin
```

**參考資料**

1. [實作 Dockerfile + flask 教學](https://www.maxlist.xyz/2020/01/11/docker-flask/)