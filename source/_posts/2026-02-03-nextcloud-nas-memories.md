---
title: "[Docker] 用NextCloud自建NAS雲端硬碟 - 安裝 Memories"
date: 2026-02-03 21:05:57
tags:
  - Docker
  - NextCloud
  - MariaDB
  - NAS
---
[入門指南](https://memories.gallery/install/)就很夠用，但筆者還是紀錄下實際安裝後遇到的過程。
<!--more-->

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

別擔心，是因為 MariaDB 服務掛掉，重啟便可。
![](/assets/mariadb_service.png)

```shell
docker exec -it nextcloud php occ memories:places-setup --transaction-size=2
```