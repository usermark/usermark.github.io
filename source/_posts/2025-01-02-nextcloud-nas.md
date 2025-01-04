---
title: "[Docker] 用NextCloud自建NAS雲端硬碟"
date: 2025-01-02 21:01:08
tags:
  - Docker
  - NextCloud
  - MariaDB
  - NAS
---
暨[上一篇](/2024/09/28/nextcloudpi-nas/)完成後，幾個月玩下來，發現自己開電腦的次數，還比接上樹莓派多，也只有想到要備份時，才會將樹莓派接上電源。
所以調整備份方式，改用電腦 Docker 架 NextCloud，外接硬碟就只拿來定時備份，而且這樣用電腦來整理重複照片也比較方便。

<!--more-->

# NextCloud

輸入指令
```bash
docker run -p 8080:80 -p 8443:443 -d -v /D/nextcloud:/var/www/html/data --name nextcloud nextcloud
```
-p 綁定連接埠port。
-d 容器啟動後會進入背景執行。
-v 用來掛載本機資料夾到容器，此處的 /D/nextcloud 便是 D:\nextcloud，是預計用來放照片的地方。冒號後面的 /var/www/html/data 為容器內對應的資料夾。
-\-name 可以為容器自訂命名。

執行後，就能連進 http://localhost:8080 查看。
![](/assets/nextcloud-init.png)

到這邊先不要急著按安裝，系統預設採用 SQLite 作為資料庫，方便快速上手，但不適用於中大型或服務站台，因此筆者改用 MariaDB。

# MariaDB

先用 root 連進去後，建立資料庫 nextcloud
```bash
mysql -u root -p
```

```sql
create database nextcloud;
```

建立使用者 nc，後面的 % 字號表示允許任何主機連線。建議換成 WSL 的區網 IP 會更安全。輸入 ipconfig 後找到 `乙太網路卡 vEthernet (WSL)` 便是。
```bash
create user 'nc'@'%' identified by 'your-password';
```

授予使用者權限
```bash
grant all on nextcloud.* to 'nc'%'%';
```

完成後，就能再次回到 http://localhost:8080 輸入對應資料安裝囉！資料庫主機記得輸入的是 WSL 的區網 IP。

# HTTPS

為了讓網路更安全，一定要進行 SSL 加密。

```bash
docker exec -it nextcloud bash
```
啟用 Apache 的 SSL 模式
```bash
a2enmod ssl
```

接下來有兩個選擇，一個是自簽憑證，一個是 Let’s Encrypt。

## 自簽憑證

```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/apache-selfsigned.key -out /etc/ssl/certs/apache-selfsigned.crt
```

修改 apache 設定檔
```bash
nano /etc/apache2/sites-available/000-default.conf
```

```xml
<VirtualHost *:443>
	ServerAdmin webmaster@localhost
	ServerName localhost
	DocumentRoot /var/www/html

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

	SSLEngine on
	SSLCertificateFile /etc/ssl/certs/apache-selfsigned.crt
	SSLCertificateKeyFile /etc/ssl/private/apache-selfsigned.key
</VirtualHost>

<VirtualHost *:80>
	ServerName localhost
	Redirect / https://localhost:8443/
</VirtualHost>
```

重啟容器後，回去刷新網頁便會自動導到 https://localhost:8443/
```bash
docker restart nextcloud
```

## Let’s Encrypt

安裝 Let’s Encrypt 客戶端 Certbot 後，執行證書安裝。注意 80 和 443 port 必須對外才會成功驗證，因為筆者使用 docker 對外的 port 分別是 8080 和 8443，有再透過 wifi 路由 port forwarding 端口轉發。
```bash
apt install certbot python3-certbot-apache
certbot --apache
```

完成後可以看下產生好的 apache 設定檔 /etc/apache2/sites-available/000-default-le-ssl.conf
```xml
<IfModule mod_ssl.c>
	<VirtualHost *:443>
		ServerName your.domain.com

		ServerAdmin webmaster@localhost
		DocumentRoot /var/www/html

		ErrorLog ${APACHE_LOG_DIR}/error.log
		CustomLog ${APACHE_LOG_DIR}/access.log combined

		SSLCertificateFile /etc/letsencrypt/live/your.domain.com/fullchain.pem
		SSLCertificateKeyFile /etc/letsencrypt/live/your.domain.com/privkey.pem
		Include /etc/letsencrypt/options-ssl-apache.conf
	</VirtualHost>
</IfModule>
```

/etc/apache2/sites-available/000-default.conf 則多了底下 3 行轉送為 https 的。
```xml
<VirtualHost *:80>
	#ServerName www.example.com

	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

	RewriteEngine on
	RewriteCond %{SERVER_NAME} =your.domain.com
	RewriteRule ^ https://%{SERVER_NAME}%{REQUEST_URI} [END,NE,R=permanent]
</VirtualHost>
```

# 優化

新增索引以改善資料庫效能
```bash
docker exec --user www-data nextcloud php occ db:add-missing-indices
```

# 常用指令

將新的照片和影片加入資料庫，等同 NextCloudPi 的 nc-scan
```bash
docker exec --user www-data nextcloud php occ files:scan --all
```

**參考資料**

1. [教學：MariaDB/MySQL常用指令操作與語法範例](https://www.pcschool.com.tw/study-technical/mysql-mariadb-sample)
2. [舊電腦變自架雲端儲存 nextcloud + samba 最詳細教學 帶你一步一步走 (Part5)](https://hkptparadise.blogspot.com/2022/12/nextcloud-samba-part5.html)