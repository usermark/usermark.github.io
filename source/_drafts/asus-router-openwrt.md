---
title: "[OpenWrt] ASUS RT-N12+ B1 路由器刷機"
tags:
  - OpenWrt
  - WPA3
---
想加強家裡的 WiFi 安全性，改用 WPA3 加密協議，發現 [WiFi安全 | 你還在用WPA2嗎？如何免費爲路由器升級WPA3加密協議](https://upsangel.com/security/how-to-free-upgrade-to-wpa3/) 這篇文章介紹，竟然可以不用花錢就升級，立馬翻出家裡許久未使用的 ASUS 路由器升級一番。
<!--more-->

查看 [OpenWrt 支援列表](https://openwrt.org/toh/start)，手邊的 ASUS RT-N12+ B1 正好有符合條件。

![](/assets/asus_router.jpg)

# 拆解

從靠近燈號的地方撬開
![](/assets/asus_router2.jpg)

找到 TTL 接腳 (紅框中)，從左往右分別是 VCC, RX ,TX, GND
![](/assets/asus_router3.jpg)

# 接上 USB 轉 TTL 的晶片

![](/assets/asus_router_ttl.jpg)

焊接技術沒有很好，用針灸的方式一根根焊在上面，再接上 USB 轉 TTL 的晶片，我用的是 CP2102。VCC 不用接，GND 接主板的 GND，RXD 接主板的 TX，TXD 接主板的 RX。
![](/assets/asus_router_ttl2.jpg)

接上電腦後，打開 Putty 建立連線，再打開路由器，就可以看到 uboot 的啟動資訊囉。
波特率多少視設備而定，若出現的是亂碼表示不對，這邊是用 57600。
![](/assets/putty_serial.png)

```sh
U-Boot 1.1.8 (Dec  7 2017 - 13:23:42)

ASUS PRODUCT bootloader version: 1.0.0.3
Board: Ralink APSoC DRAM:  32 MB
ASUS ASUS PRODUCT gpio init : reset pin
flash manufacture id: ef, device id 40 17
find flash: W25Q64BV
raspi_read: from:40035 len:1
raspi_read: from:40036 len:1
raspi_read: from:30000 len:1000
Maximum malloc length: 1024 KBytes
mem_malloc_start/brk/end: 0x81eb3000/81eb5000/81fb4000
*** Warning - bad CRC, using default environment

============================================
Ralink UBoot Version: 4.3.0.0
--------------------------------------------
ASIC 7628_MP (Port5<->None)
DRAM component: 256 Mbits DDR, width 16
DRAM bus: 16 bit
Total memory: 32 MBytes
Flash component: SPI Flash
Date:Dec  7 2017  Time:13:23:42
============================================
icache: sets:512, ways:4, linesz:32 ,total:65536
dcache: sets:256, ways:4, linesz:32 ,total:32768
RESET MT7628 PHY!!!!!!
Please choose the operation:
   0: Load System code then write to Flash via Serial.
   1: Load System code to SDRAM via TFTP.
   2: Load System code then write to Flash via TFTP.
   3: Boot System code via Flash (default).
   4: Entr boot command line interface.
   7: Load Boot Loader code then write to Flash via Serial.
   9: Load Boot Loader code then write to Flash via TFTP.                    0

3: System Boot System code via Flash.
raspi_read: from:4018a len:4

ASUS PRODUCT bootloader version: 1.0.0.3
raspi_read: from:40004 len:6
MAC Address: XX:XX:XX:XX:XX:XX
raspi_read: from:40004 len:6

## Checking 1st firmware at bc050000 ...
raspi_read: from:50000 len:40
   Image Name:
   Image Type:   MIPS Linux Kernel Image (lzma compressed)
   Data Size:    7628444 Bytes =  7.3 MB
   Load Address: 80000000
   Entry Point:  8000c150
raspi_read: from:50040 len:74669c
   Verifying Checksum ... OK
   Uncompressing Kernel Image ... OK
## Giving linux ramsize: 33554432 (32 MB)

Starting kernel ...
```

# 燒錄

重啟路由器，並選擇 2: Load System code then write to Flash via TFTP.
```sh
You choosed 2


raspi_read: from:40004 len:6


2: System Load System code then write to Flash via TFTP.
 Warning!! Erase Linux in Flash then burn new one. Are you sure?(Y/N)
```

**參考資料**

1. [\[分享\] ASUS RT-AC51U 無線路由器 拆解](https://x14006.pixnet.net/blog/post/231291799)
2. [网件(NETGEAR)wndr4300 TTL线刷救砖记](https://www.spirithy.com/2016/09/18/netgear4300-ttl-flash/)
3. [使用TFTP和TTL在U-BOOT烧写路由内核 刷机 救砖 基于IPQ4019](http://www.elelab.net/use-tftpd32-ttl-write-u-boot.html)
4. [XiaomiRouter自学之路(08-U-boot启动数值具体说明)](https://www.openwrt.pro/post-239.html)
4. [刷 OpenWRT 韌體刷機記錄 – 華碩 ASUS RT-AC58U 路由器](https://blog.jks.coffee/flashing-openwrt-firmware-asus-rt-ac58u-router/)