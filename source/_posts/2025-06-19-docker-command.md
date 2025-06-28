---
title: '[Docker] 常用指令'
date: 2025-06-19 16:55:32
tags:
- Docker
---
整理常用的指令。
<!--more-->

查看目前 images
```bash
docker images
```

查看目前運行的 container
```bash
docker ps
```

查看目前全部的 container
```bash
docker ps -a
```

新建並啟動 container
```bash
docker run -p 80:80 -d --name my_container nginx
```
-p 綁定連接埠port。
-d 容器啟動後會進入背景執行。
-\-name 可以為容器自訂命名。

進入 container
```bash
docker exec -it my_container bash
```

複製本地文件到 container
```bash
docker cp local_file.txt my_container:/app/
```

複製 container 內文件到本地 D:\\temp
```bash
docker cp my_container:/app/data.txt /D/temp/
```

根據 Dockerfile 打包成 image 
```bash
docker image build -t asia-east1-docker.pkg.dev/xxx/hub/frps:v1 .
```

上傳至 Artifact Registry
```bash
docker push asia-east1-docker.pkg.dev/xxx/hub/frps:v1
```

**參考資料**
1. [實作 Dockerfile + flask 教學](https://www.maxlist.xyz/2020/01/11/docker-flask/)