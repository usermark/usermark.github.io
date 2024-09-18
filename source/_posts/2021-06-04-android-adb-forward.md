---
layout: article
title: "[Android] adb forward"
date: 2021-06-04 17:34
tags: Android
---

了解 adb 背後原理和運用。

<!--more-->

# adb forward

轉發 PC 端 4000 端口到手機的 9999 端口

```
adb forward tcp:4000 tcp:9999
```

查看是否轉發成功

```
adb forward --list
```

## 範例

Android 建立 server socket

```java
private static class SocketThread extends Thread {
    ServerSocket serverSocket;

    @Override
    public void run() {
        byte[] buffer = new byte[1024];
        try {
            serverSocket = new ServerSocket(9999);
            Log.d("Mike", "socket open");
            Socket clientSocket = serverSocket.accept(); // 此處會阻塞等待
            Log.d("Mike", "socket accept");

            InputStream is = clientSocket.getInputStream();
            OutputStream os = clientSocket.getOutputStream();
            while (!isInterrupted()) {
                int len = is.read(buffer);
                if (len > 0) {
                    os.write(("echo " + new String(buffer, 0, len)).getBytes());
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        Log.d("Mike", "socket end");
    }

    public void close() {
        if (serverSocket != null) {
            try {
                interrupt();
                serverSocket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

PC 端用 python 建立 client socket

```python
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect((socket.gethostname(), 4000))
client.send(b'Hello')
print(client.recv(1024))
```

# adb 通道

adb server 啟動以後，在本地的 5037 端口監聽。adb client 通過本地的隨機端口與 5037 端口建立連接。
在這個通道上，client 向 server 發送的命令都遵循如下格式：

1. 命令的長度(Length)，由四位的十六進製表示
2. 實際的命令(Payload)，通過 ASCII 編碼

譬如，查看 adb 當前的版本，client 會發起如下命令：

```
000Chost:version
```

000C 表示”host:version”這條命令的長度為 12 個字節。命令中使用了 host 前綴，目的是為了區分其他類型的命令，host 前綴的命令可以理解為只在 client 與 server 這個數據通道上存在。

server 收到 client 的請求後，返回的數據遵循如下格式：

1. 如果成功，則返回四個字節的字符串”OKAY”
2. 如果失敗，則返回四個字節的字符串”FAIL”和出錯原因
3. 如果異常，則返回錯誤碼

當 Client 收到 Server 返回的”OKAY”後，就可以發繼續發起其他操作命令了。

當與 server 建立第一個通道的連接後，需要向 server 發送 transport 命令，表示接下來，要與 adbd 進行數據傳輸。

```
host:transport:<serial-number>
```

當 server 返回“OKAY”後，client 後續發送的數據，就直接傳輸到 adbd 了。

## 範例

Android 建立 server socket

```java
private static class SocketThread extends Thread {
    LocalServerSocket serverSocket;

    @Override
    public void run() {
        byte[] buffer = new byte[1024];
        try {
            serverSocket = new LocalServerSocket("test");
            Log.d("Mike", "socket open");
            LocalSocket clientSocket = serverSocket.accept(); // 此處會阻塞等待
            Log.d("Mike", "socket accept");

            InputStream is = clientSocket.getInputStream();
            OutputStream os = clientSocket.getOutputStream();
            while (!isInterrupted()) {
                int len = is.read(buffer);
                if (len > 0) {
                    os.write(("echo " + new String(buffer, 0, len)).getBytes());
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
        Log.d("Mike", "socket end");
    }

    public void close() {
        if (serverSocket != null) {
            try {
                interrupt();
                // 與ServerSocket不同，close不會中斷accept，需用假的連上來跳過
                LocalSocket mockSocket = new LocalSocket();
                mockSocket.connect(serverSocket.getLocalSocketAddress());
                serverSocket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

PC 端用 python 建立 client socket

```python
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# 當socket關閉後，本地端用於該socket的端口就可以立刻被重用
client.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
client.connect((socket.gethostname(), 5037))

transport = 'host:transport:192.168.2.131:5555'
client.send(bytes('%0.4X' % len(transport) + transport, 'ascii'))
print(client.recv(4))
client.send(b'0012localabstract:test')
print(client.recv(4))
client.send(b'Hello')
print(client.recv(1024))
```

參考<https://duanqz.github.io/2015-05-21-Intro-adb>
