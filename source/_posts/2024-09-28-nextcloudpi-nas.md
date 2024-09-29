---
title: "[Raspberry Pi] 用NextCloudPi自建NAS雲端硬碟"
tags:
  - Raspberry Pi
  - NextCloudPi
  - NAS
date: 2024-09-28 23:38:01
---

想擁有自己的 NAS 雲端硬碟，資料不想放別人家被看光光，卻又不想花大錢，雖說 QNAP TS-133 也才 4 千塊。
但自己打造樂趣無窮，決定先使用樹莓派和 1TB 外接 USB 硬碟體驗下陽春版的私有雲。

<!--more-->

# NextCloudPi

至 <https://github.com/nextcloud/nextcloudpi/releases> 下載最新的映像檔，我使用的是 Pi4。
使用樹莓派官方提供的 Raspberry Pi Imager 做燒錄。
![](/assets/rpi_imager.png)

燒錄完成後，插入 SD 卡並啟動樹莓派，先不要外接硬碟，稍後依指示再接上。
另外網路部分我是接實體網路線，與個人電腦在同一網段。
![](/assets/home_net_pi.png)

等待一段時間便可用瀏覽器打開 https://nextcloudpi.local
第一次會看到 Activation 的畫面，記住兩組帳密後，按下 Activate 開啟初始化設定。
畫面轉到後台管理 https://nextcloudpi.local:4443 後，使用 ncp 第一組帳密登入。
![](/assets/ncp_wizard.png)

依照初始化精靈的指示做設定，在詢問是否會使用外接硬碟來儲存資料時，我選 Yes。
![](/assets/ncp_wizard_usb.png)

這時才接上外接硬碟，並做格式化，注意資料會被刪除。因為一般外接硬碟都是 NTFS 檔案格式，並不支援 Linux 所需的權限控管，要格式化成 Btrfs 才行。
另外，前面啟動不先接上是因為 NextCloud 預設會自動掛載外接硬碟至 /media/USBdrive，這時就無法格式化。
初始化精靈完成後，可檢查下 data disk usage 的硬碟空間是否為 1TB。
![](/assets/ncp_system_info.png)

初步架設到這邊就都完成了，可以連進 https://nextcloudpi.local 使用 ncp 第二組帳密登入體驗囉!

# 建立使用者

登入後，習慣會先建立群組和使用者 (或家庭成員)，與 ncp 管理者權限做區分。
![](/assets/ncp_user.png)

在檔案頁籤會看到一些系統內建的檔案，但都還沒有個人的照片和檔案。

# Samba

回到後台管理 4443 開啟 samba。目的是透過個人電腦先將 Google Photo 和 Line 相簿備份回 NextCloud。
![](/assets/ncp_samba.png)

# 備份 Google Photo 至 NextCloud

下載 [rclone](https://rclone.org/downloads/)，並申請一組自己 Google 帳號的 OAuth Clinet ID 來使用 rclone。

輸入下列指令進行一系列設定，包含 client_id 和 client_secret

```sh
rclone config
```

設定好後輸入指令查看所有相簿，其中 remote 是前面設定的 name

```sh
rclone lsd remote:album
```

這裡要特別注意的是同張照片會因不同分類，而重複存在不同目錄，所以不要全部都抓，參考 <https://rclone.org/googlephotos/> 的說明。
這邊採用的是按年分類，並複製照片和影片至指定目錄。

```sh
rclone copy remote:media/by-year /path/to/images -P
```

等全部下載完，就可以透過 Samba 備份進 NextCloud。這時進去 https://nextcloudpi.local 還不會看到剛才上傳的照片和影片，要先在後台管理 4443 執行 nc-scan ，會將新的照片和影片加入資料庫，這樣 NextCloud 就有 Google Photo 所有照片資料囉！
![](/assets/ncp_scan.png)

# 修正影片沒有預覽圖(縮圖)的問題

上傳的影片預設是沒有預覽圖的，要一個個點開才會知道很不方便
參考 <https://shengyu7697.github.io/nextcloud/>，解法如下。

輸入指令安裝

```sh
sudo apt-get install ffmpeg
```

在 config.php 新增設定
sudo nano /var/www/nextcloud/config/config.php

```php
'enable_previews' => true,
'enabledPreviewProviders' => [
    'OC\Preview\Movie',
    'OC\Preview\PNG',
    'OC\Preview\JPEG',
    'OC\Preview\GIF',
],
```

# 修正 HEIC 沒有預覽圖(縮圖)的問題

輸入指令確認 imagick 模組是否存在
```sh
php -m | grep -i imagick
```

沒有的話就安裝並重啟 apache2
```sh
sudo apt-get install php-imagick
sudp service apache2 reload
```

在 config.php 新增設定
sudo nano /var/www/nextcloud/config/config.php

```php
'enable_previews' => true,
'enabledPreviewProviders' => [
    'OC\Preview\Movie',
    'OC\Preview\HEIC',
    'OC\Preview\PNG',
    'OC\Preview\JPEG',
    'OC\Preview\GIF',
],
```

# NextCloud 公開對外

![](/assets/freedns.png)

於 [FreeDNS](https://freedns.afraid.org/) 申請域名來用，在後台管理 4443 設定 freeDNS 即可。
![](/assets/ncp_freedns.png)

接著執行 letsencrypt 申請 SSL 憑證，才不會一直看到安全性警告。這裡 80 和 443 port 必須對外才會成功驗證，若是透過 wifi 路由上網的，要去設定 port forwarding 端口轉發。
![](/assets/ncp_ssl.png)

# 手機 APP 自動上傳備份照片

安裝 iOS/Android 的 APP 程式，來連接 Nextcloud。成功登入後，可至更多 > 設定 > 自動上傳，將自動上傳照片/影片功能打開。預設只會上傳後續新增的照片，可視情況決定是否打開上傳整個相機膠捲。
因為我平時都用 Google Photo 備份，前面已將資料遷移回來，所以也就不需要上傳舊的資料。

# 備份

在後台管理 4443 執行 nc-backup ，便可產生備份檔。

{% note warning %}
切記備份檔路徑不可以選擇底下有 ncdata 資料夾的，因為備份過程該目錄會變成唯讀狀態。

```sh
Running nc-backup
Installing template 'ncp-metrics.cfg.sh'...
check free space...
Maintenance mode enabled
backup database...
backup files...
tar: /media/myCloudDrive/ncp-backups/nextcloud-bkp_20240921_1726902238.tar: Cannot write: Read-only file system
tar: Error is not recoverable: exiting now
error generating backup
{"reqId":"lp43MaIygxNeP3vd4WWI","level":2,"time":"2024-09-21T07:06:01+00:00","remoteAddr":"","user":"--","app":"no app in context","method":"","url":"--","message":"Temporary directory /media/myCloudDrive/ncdata/data/tmp is not present or writable","userAgent":"--","version":"29.0.4.1","data":[]}
{"reqId":"lp43MaIygxNeP3vd4WWI","level":3,"time":"2024-09-21T07:06:01+00:00","remoteAddr":"","user":"--","app":"PHP","method":"","url":"--","message":"fopen(/media/myCloudDrive/ncdata/data/data_dir_writability_test_66ee70595461e.tmp): Failed to open stream: Read-only file system at /var/www/nextcloud/lib/private/legacy/OC_Util.php#572","userAgent":"--","version":"29.0.4.1","data":{"app":"PHP"}}
Your data directory is not writable.
Permissions can usually be fixed by giving the web server write access to the root directory. See https://docs.nextcloud.com/server/29/go.php?to=admin-dir_permissions.
```
{% endnote %}

# 異地備份

目的備份電腦為 windows 所以無法使用 rsync，rsync 只能在 linux 之間使用，不得不提到強大的 rclone。
先在目的備份電腦的指定目錄打開 sftp 後，至後台管理 4443 打開 SSH。

![](/assets/ncp_ssh.png)

成功 ssh 登入 nextcloudpi.local 後，輸入下列指令進行 rclone 的安裝並設定遠端 sftp。
```sh
sudo -v ; curl https://rclone.org/install.sh | sudo bash
rclone config
```

使用 rclone 存取 ncdata 目錄需要有 www-data 的群組權限，否則會有該錯誤。
```sh
ERROR : Local file system at /media/myCloudDrive/ncdata/data/ncp/files: error reading source root directory: directory not found
```

輸入指令加入群組，記得重新登入才會生效。
```sh
sudo gpasswd -a {your-ssh-user} www-data
```

就可以用 rclone sync 開始同步目的備份電腦的資料囉！
```sh
rclone sync -P /media/myCloudDrive/ncdata/data/ncp/files remote:
```

# 總結

上述步驟完成後，陽春版的私有雲便打造完成。
出遊時手機拍的照片便會透過 Nextcloud APP 即時備份至雲端硬碟，回到家打開電腦 ssh 進樹莓派做異地備份，更進階的玩法可以用排程實現。
手機一份、雲端硬碟一份，需要時電腦才連上去備份一份，算有勉強符合 321 備份原則了。

**參考資料**

1. [\[教學\] 自己雲端自己管！ | 以 NextCloud 與 Raspberry Pi 自建 NAS 雲端硬碟](https://vocus.cc/article/63a3f27cfd897800011b0a0a)
2. [使用 rclone 同步 Google Drive 檔案](https://www.ichiayi.com/tech/rclone)
