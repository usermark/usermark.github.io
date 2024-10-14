---
title: "[Web] 低成本架站 part2"
date: 2024-10-09 17:37:39
tags:
  - OpenWrt
  - 小烏龜
---

完成 OpenWrt 刷機後，發現其支援 uHTTPd 是可以用來架站的，且不用另外安裝。

<!--more-->

![](/assets/home_net2.png)

附上網路架構圖，與[上次](/2022/11/13/low-cost-host/)不同的是這次是用 OpenWrt 來當 Server 架站。
使用 SSH 連進 OpenWrt 後，查看 www 目錄

```sh
ls /www
```

可以看到底下有 index.html，這便是原始 OpenWrt 的後臺管理介面。
新增 welcome 目錄以及底下的 index.html

```sh
cd /www
mkdir welcome
vim welcome/index.html
```

內容如下

```
<h1>Welcome to my site</h1>
```

# 設定 uhttpd

編輯 uhttpd 設定檔

```sh
vim /etc/config/uhttpd
```

詳細設定參考 <https://openwrt.org/docs/guide-user/services/webserver/uhttpd>

基本設定長這樣
```
config uhttpd 'main'
        list listen_http '0.0.0.0:80'
        list listen_http '[::]:80'
        list listen_https '0.0.0.0:443'
        list listen_https '[::]:443'
        option home '/www'
```

再新增一組剛才建立的 welcome

```
config uhttpd 'welcome'
        list listen_http '0.0.0.0:8000'
        option home '/www/welcome'
```

輸入下方指令重啟服務

```sh
/etc/init.d/uhttpd restart
```

# 防火牆對外

還需要開啟防火牆才能對外，Network > Firewall > Traffic Rules

![](/assets/openwrt_firewall.png)

表示允許外部 wan 透過 IPv4 TCP 協定存取該設備的 8000 port
