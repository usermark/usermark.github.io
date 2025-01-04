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
docker run -p 7500:7500 -p 6001:6001 -d -v data:/etc/frp --name frps snowdreamtech/frps
```
-p 綁定連接埠port。7500 port是給frp網頁使用，6001 port用來作為對外的窗口，待會會用到。
-d 容器啟動後會進入背景執行。
-v 用來掛載資料卷到容器，此處的data便是剛建立的。冒號後面的/etc/frp為容器內對應的資料夾。
-\-name 可以為容器自訂命名。

![](/assets/docker_frps.png)

執行後，會發現裡面多了一個frps.toml
![](/assets/docker_volumes_2.png)

進入frps，輸入vi指令編輯
```sh
docker exec -it frps sh
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
docker run -d -v data:/etc/frp --name frpc snowdreamtech/frpc
```
容器間的port是互通的，不需要用-p綁定。

進入frps，輸入vi指令編輯
```sh
docker exec -it frps sh
vi /etc/frp/frpc.toml
```

貼上內容並儲存
```
serverAddr = "172.17.0.2"
serverPort = 7000
auth.token = "12345678"

[[proxies]]
name = "test-tcp"
type = "tcp"
localIP = "127.0.0.1"
localPort = 4000
remotePort = 6001
```
serverAddr 要是frps的IP。
重新執行，結果如下
![](/assets/frpc.png)

# 測試

進入frpc做測試，並監聽4000 port，等待接收。
```sh
docker exec -it frpc sh
echo "hello" | nc -l -p 4000
```

另外打開cmd視窗，輸入
```sh
telnet localhost 6001
```
若有收到回應hello，表示成功囉!
流程大致是telnet打向本地主機的6001 port，會轉送給frps主機的6001 port。根據frpc.toml配置會轉送給frpc主機的4000 port。
這樣既使frpc主機在內網，也能對外開放其服務。

**參考資料**
1. [FRP 内网穿透工具部署](https://blog.csdn.net/networken/article/details/134994593)
2. [以Docker架設frp(fast reverse proxy)流程](https://suyenting.github.io/post/docker-frp/)