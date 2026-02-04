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

# 支援 HEIC 和影片格式

進入 container

```shell
docker exec -it bash 
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
nano config/config.php

```php
'enable_previews' => true,
'enabledPreviewProviders' => array(
    0 => 'OC\\Preview\\Movie',
    1 => 'OC\\Preview\\HEIC',
    2 => 'OC\\Preview\\Image',
),
```

# Reverse Geocoding (Places)

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

接著改用 occ 指令抓取 Reverse Geocoding (Places)
```shell
docker exec -it nextcloud php occ memories:places-setup --transaction-size=5
```

還是失敗的話，要