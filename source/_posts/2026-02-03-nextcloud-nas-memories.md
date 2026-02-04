---
title: "[Docker] 用NextCloud自建NAS雲端硬碟 - 安裝 Memories"
date: 2026-02-03 21:05:57
tags:
  - Docker
  - NextCloud
  - MariaDB
  - NAS
---
[Memories 入門指南](https://memories.gallery/install/)就很夠用，但筆者還是紀錄下實際安裝過程中遇到的問題。
<!--more-->
初步安裝好 Memories 後，進入 NextCloud >「管理設定」>「回憶」

# 支援 HEIC 和影片格式

進入 container

```shell
docker exec -it nextcloud bash 
```

輸入指令安裝 ffmpeg

```shell
apt-get update
apt-get install ffmpeg
```

輸入指令安裝 nano

```shell
apt-get install nano
```

在 config.php 新增設定
nano /var/www/html/config/config.php

```php
'enabledPreviewProviders' => 
array(
  0 => 'OC\\Preview\\Movie',
  1 => 'OC\\Preview\\HEIC',
  2 => 'OC\\Preview\\Image',
),
```

# 反向地理編碼 Reverse Geocoding (Places)

選擇「下載地圖資料庫」
![](/assets/memories_places.png)

挖~ 出現這行錯誤

```
ERROR: Failed to insert polygon 14590532_0 (An exception occurred while executing a query: SQLSTATE[HY000]: General error: 2006 MySQL server has gone away 
Failed: An exception occurred while executing a query: SQLSTATE[HY000]: General error: 2006 MySQL server has gone away
```

就連網站都掛了

```
Internal Server Error

The server encountered an internal error and was unable to complete your request.
Please contact the server administrator if this error reappears multiple times, please include the technical details below in your report.
More details can be found in the server log.
```

別擔心，是因為 MariaDB 服務掛掉，重啟便可。再次刷新網站就活過來了。
![](/assets/mariadb_service.png)

接著改用 occ 指令抓取地圖資料庫
```shell
docker exec -it nextcloud php occ memories:places-setup --transaction-size=5
```

還是失敗的話，要修改 max_allowed_packet，可用下列指令查看，預設是 16M
```shell
SHOW VARIABLES LIKE 'max_allowed_packet';
```

修改 C:\Program Files\MariaDB 11.6\data\my.ini，改成 512M
```ini
[mysqld]
datadir=C:/Program Files/MariaDB 11.6/data
port=3306
innodb_buffer_pool_size=4048M
max_allowed_packet=512M
[client]
port=3306
plugin-dir=C:\Program Files\MariaDB 11.6/lib/plugin
```

重啟 MariaDB 服務後再次輸入前個指令，確認是否修改成功。

```shell
docker exec -it nextcloud php occ memories:index
```

再次選擇「下載地圖資料庫」，等待一段時間就會成功。