---
title: '[Docker] 入門-frp實戰'
date: 2024-03-16 15:41:46
tags:
- Docker
- frp
---
拿frp做練習，重新認識docker。
<!--more-->
[frp](https://github.com/fatedier/frp)為內網穿透工具，可用來將本地網站或服務對外。
首先建立Volumes(資料卷)，用來存放資料，名稱為data。
![](/assets/docker_volumes.png)
建立好後會發現裡面什麼都沒有，先不管它，繼續往下。

# frps

輸入指令
```bash
docker run -p 7000:7000 -p 7500:7500 -d -v data:/etc/frp --name frps snowdreamtech/frps
```
-p 綁定連接埠port。
-d 容器啟動後會進入背景執行。
-v 用來掛載資料卷到容器，此處的data便是剛建立的。冒號後面的/etc/frp為容器內對應的資料夾。
--name 可以為容器自訂命名。

![](/assets/docker_frps.png)

執行後，會發現裡面多了一個frps.toml
![](/assets/docker_volumes_2.png)

切換到Exec頁籤，輸入vi指令編輯
```sh
vi /etc/frp/frps.toml
```

輸入i可進入編輯模式，貼上內容
```
bindPort = 7000
webServer.addr = "0.0.0.0"
webServer.port = 7500
webServer.user = "admin"
webServer.password = "admin"
vhostHTTPPort = 8080
vhostHTTPSPort = 8443
auth.token = "12345678"
```
編輯好後，按ESC跳出編輯模式，接著輸入:wq 儲存並離開。

重新執行，結果如下
![](/assets/docker_frps_2.png)

會看到port 7500已打開，可瀏覽http://localhost:7500
![](/assets/frps.png)

# frpc

輸入指令
```
docker run --net=host -d -v data:/etc/frp --name frpc snowdreamtech/frpc
```

切換到Exec頁籤，輸入vi指令編輯
```sh
vi /etc/frp/frpc.toml
```

貼上內容並儲存
```
serverAddr = "127.0.0.1"
serverPort = 7000
auth.token = "12345678"

[[proxies]]
name = "test-tcp"
type = "tcp"
localIP = "127.0.0.1"
localPort = 4000
remotePort = 6001
```

**參考資料**
1. [FRP 内网穿透工具部署](https://blog.csdn.net/networken/article/details/134994593)
2. [以Docker架設frp(fast reverse proxy)流程](https://suyenting.github.io/post/docker-frp/)