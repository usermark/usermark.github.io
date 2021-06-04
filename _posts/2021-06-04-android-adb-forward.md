---
layout: post
title: '[Android] adb forward'
date: 2021-06-04 17:34
categories: 
---
# adb forward
轉發PC端4000端口到手機的9999端口
```
adb forward tcp:4000 tcp:9999
```
查看是否轉發成功
```
adb forward --list
```

## 範例
Android建立server socket
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

PC端用python建立client socket
```python
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect((socket.gethostname(), 4000))
client.send(b'Hello')
print(client.recv(1024))
```

# adb通道
adb server啟動以後，在本地的5037端口監聽。adb client通過本地的隨機端口與5037端口建立連接。
在這個通道上，client向server發送的命令都遵循如下格式：
1. 命令的長度(Length)，由四位的十六進製表示
2. 實際的命令(Payload)，通過ASCII編碼

譬如，查看adb當前的版本，client會發起如下命令：
```
000Chost:version
```
000C表示”host:version”這條命令的長度為12個字節。命令中使用了host前綴，目的是為了區分其他類型的命令(後面還會看到shell前綴的命令)，host前綴的命令可以理解為只在client與server這個數據通道上存在。

server收到client的請求後，返回的數據遵循如下格式：
1. 如果成功，則返回四個字節的字符串”OKAY”
2. 如果失敗，則返回四個字節的字符串”FAIL”和出錯原因
3. 如果異常，則返回錯誤碼

當Client收到Server返回的”OKAY”後，就可以發繼續發起其他操作命令了。

當與server建立第一個通道的連接後，需要向server發送transport命令，表示接下來，要與adbd進行數據傳輸。
```
host:transport:<serial-number>
```
當server返回“OKAY”後，client後續發送的數據，就直接傳輸到adbd了。

## 範例
Android建立server socket
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

PC端用python建立client socket
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

參考連結：[https://duanqz.github.io/2015-05-21-Intro-adb]()