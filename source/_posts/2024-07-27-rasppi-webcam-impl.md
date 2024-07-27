---
title: '[Raspberry Pi] WebCam實作'
date: 2024-07-27 15:20:37
tags:
- Raspberry Pi
---
使用手邊已有的上古USB鏡頭做練習，解析度只有320x240。
<!--more-->
![](/assets/cubeternet.jpg)

檢查WebCam是否已接上，輸入指令後可以看到 Cubeternet GL-UPC822 UVC WebCam
```sh
$ lsusb
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 004: ID 046d:c52b Logitech, Inc. Unifying Receiver
Bus 001 Device 003: ID 1e4e:0102 Cubeternet GL-UPC822 UVC WebCam
Bus 001 Device 002: ID 2109:3431 VIA Labs, Inc. Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

使用 Video4Linux 指令查看裝置支援的格式。
```sh
$ v4l2-ctl --list-formats
ioctl: VIDIOC_ENUM_FMT
        Type: Video Capture

        [0]: 'YUYV' (YUYV 4:2:2)
```

使用 MJPG-Streamer 建立影片串流伺服器
```sh
$ sudo apt-get install cmake libjpeg-dev git
$ git clone https://github.com/jacksonliam/mjpg-streamer.git
$ cd mjpg-streamer/mjpg-streamer-experimental
```

編譯後安裝起來
```sh
$ make
$ sudo make install
$ which mjpg_streamer
/usr/local/bin/mjpg_streamer
```

啟動串流功能
```sh
$ mjpg_streamer -i "input_uvc.so -n -f 30 -r 320x240" -o "output_http.so -p 8888 -w /usr/local/share/mjpg-streamer/www"
MJPG Streamer Version: git rev: 310b29f4a94c46652b20c4b7b6e5cf24e532af39
 i: Using V4L2 device.: /dev/video0
 i: Desired Resolution: 320 x 240
 i: Frames Per Second.: 30
 i: Format............: JPEG
 i: TV-Norm...........: DEFAULT
 i: Could not obtain the requested pixelformat: MJPG , driver gave us: YUYV
    ... will try to handle this by checking against supported formats.
    ... Falling back to YUV mode (consider using -yuv option). Note that this requires much more CPU power
 i: FPS coerced ......: from 30 to 4
 o: www-folder-path......: /usr/local/share/mjpg-streamer/www/
 o: HTTP TCP port........: 8888
 o: HTTP Listen Address..: (null)
 o: username:password....: disabled
 o: commands.............: enabled
```
會看到一段錯誤表示鏡頭不支援MJPG，暫不理會。

打開瀏覽器查看
http://localhost:8888

![](/assets/pi_with_webcam.png)

**參考資料**

1. [Secure Webcam streaming with MJPG-Streamer on a Raspberry Pi](https://www.sigmdel.ca/michel/ha/rpi/streaming_en.html)
2. [使用 MJPG-Streamer 進行串流](https://dic.vbird.tw/iot_pi/unit13.php#13.3)