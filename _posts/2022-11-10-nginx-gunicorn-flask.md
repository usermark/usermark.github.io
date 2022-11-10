---
layout: article
title: '[Python] Mac 安裝 Nginx + Gunicorn + Flask'
date: 2022-11-10 09:36
tags: Python Flask Nginx
---
<!--more-->
# Flask Hello World程式
建立hello資料夾，底下新增hello_world.py
```python
from flask import Flask
 
app = Flask(__name__)
 
@app.route('/')
def index():
    return 'Hello World!'

if __name__ == '__main__':
    app.run()
```

安裝flask
```shell
pipenv install flask
pipenv run python3 hello_world.py
```

結果如下
```shell
 * Serving Flask app 'hello_world'
 * Debug mode: off
WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
 * Running on http://127.0.0.1:5000
Press CTRL+C to quit
```

# 使用Gunicorn

安裝gunicorn
```shell
pipenv install gunicorn
pipenv run gunicorn hello_world:app
```

結果如下
```shell
[2022-11-10 22:40:45 +0800] [37663] [INFO] Starting gunicorn 20.1.0
[2022-11-10 22:40:45 +0800] [37663] [INFO] Listening at: http://127.0.0.1:8000 (37663)
[2022-11-10 22:40:45 +0800] [37663] [INFO] Using worker: sync
[2022-11-10 22:40:45 +0800] [37671] [INFO] Booting worker with pid: 37671
```

# 使用Nginx反向代理本地的Gunicorn服務

安裝Nginx
```shell
brew update
brew install nginx
```

使用以下指令得知相關配置
```shell
brew info nginx
```

結果如下
```shell
==> nginx: stable 1.23.2 (bottled), HEAD
HTTP(S) server and reverse proxy, and IMAP/POP3 proxy server
https://nginx.org/
/usr/local/Cellar/nginx/1.23.2 (26 files, 2.2MB) *
  Poured from bottle on 2022-11-09 at 20:34:30
From: https://github.com/Homebrew/homebrew-core/blob/HEAD/Formula/nginx.rb
License: BSD-2-Clause
==> Dependencies
Required: openssl@1.1 ✔, pcre2 ✔
==> Options
--HEAD
	Install HEAD version
==> Caveats
Docroot is: /usr/local/var/www

The default port has been set in /usr/local/etc/nginx/nginx.conf to 8080 so that
nginx can run without sudo.

nginx will load all files in /usr/local/etc/nginx/servers/.

To restart nginx after an upgrade:
  brew services restart nginx
Or, if you don't want/need a background service you can just run:
  /usr/local/opt/nginx/bin/nginx -g daemon off;
==> Analytics
install: 49,084 (30 days), 114,716 (90 days), 459,040 (365 days)
install-on-request: 49,004 (30 days), 114,535 (90 days), 458,238 (365 days)
build-error: 0 (30 days)
```

將hello資料夾移至 /usr/local/var/www 底下

修改Nginx設定檔 /usr/local/etc/nginx/nginx.conf
```
server {
    listen       80;
    server_name  localhost;

    location / {
        root   hello;
        index  index.html index.html;
        proxy_pass http://127.0.0.1:8000;
        proxy_redirect off;
        proxy_set_header Host $host:80;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

檢查配置是否正確
```shell
nginx -t
```

重新啟動Nginx
```shell
nginx -s reload
```

**參考資料**
1. [Nginx + Gunicorn + Flask 整合配置](https://www.gushiciku.cn/pl/pNKQ/zh-tw)
2. [Mac 安裝 Nginx](https://iter01.com/521092.html)