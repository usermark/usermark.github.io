---
layout: article
title: '[Raspberry Pi] Android TV'
date: 2023-07-30 08:16:53
tags:
- Raspberry Pi
- Android
---
終於等到樹莓派降價，購入的是Raspberry Pi 4B，趕緊先裝個Android TV看看成果。
<!--more-->

# 下載LineageOS

首先進入<https://konstakang.com/devices/rpi4/>取得作者已處理好，鏡像出來的映像檔。選擇有包含Android TV的，LineageOS 20 Android TV - KonstaKANG (Android 13)
![](/assets/lineageOS_20.png)

下載lineage-20.0-20230505-UNOFFICIAL-KonstaKANG-rpi4-atv.zip
![](/assets/lineageOS_20_2.png)

因為是鏡像出來的映像檔，若使用的是超過8GB的SD card，會有未配置到的空間，要下載lineage-20.0-rpi-resize.zip擴充。
![](/assets/lineageOS_20_resize.png)

本次使用的是64GB的SD card
![](/assets/lineageOS_sdcard_space.png)

下載MindTheGapps-13.0.0-arm64-ATV-20230321_142647.zip，讓其支援Play商店。
![](/assets/lineageOS_20_google_play.png)

# 燒錄映像檔

使用的燒錄工具為樹莓派官方提供的Raspberry Pi Imager
![](/assets/rpi_imager.png)

選好儲存卡後，點選 選擇操作系統 > 使用自定義鏡像
![](/assets/rpi_imager_custom.png)

點選燒錄後，等待完成
![](/assets/rpi_imager_burn.png)

![](/assets/rpi_imager_finish.png)

# 準備USB隨身碟

另外準備一個USB隨身碟，放入剛下載好的resize和MindTheGapps的zip檔。
![](/assets/lineageOS_usb.png)

# 打開LineageOS

將SD card和USB隨身碟放入樹莓派後，接上鍵盤和滑鼠，打開它。

至"About"，打開"開發人員模式"
![](/assets/lineageOS_1.png)

至"Buttons"，打開"Advanced reboot"
![](/assets/lineageOS_2.png)

會發現"Restart"多了"Recovery"選項，點它
![](/assets/lineageOS_3.png)

重啟後會進入此畫面，點選"Install"
![](/assets/lineageOS_4.png)

點選"Select Storage"，選擇USB隨身碟
![](/assets/lineageOS_5.png)

選擇lineage-20.0-rpi-resize.zip，接著按下"Add more Zips"
![](/assets/lineageOS_6.png)

選擇MindTheGapps-13.0.0-arm64-ATV-20230321_142647.zip，滑動右側的滾動條
![](/assets/lineageOS_7.png)

等待安裝完成
![](/assets/lineageOS_8.png)

安裝完成後，選擇"Wipe Dalvik"
![](/assets/lineageOS_9.png)

滑動"Swipe to Wipe"
![](/assets/lineageOS_10.png)

等待Wipe完成後，按下"Reboot System"
![](/assets/lineageOS_11.png)

重啟後查看儲存空間，已擴充到59GB
![](/assets/lineageOS_12.png)

應用程式也可看到Play商店
![](/assets/lineageOS_13.png)

# 問題排除

但要登入時，會遇到沒有網路連線的問題，但實際是有連上網路的
![](/assets/lineageOS_14.png)

試了幾次操作，發現要選擇"Wipe"
![](/assets/lineageOS_4.png)

滑動右側的滾動條
![](/assets/lineageOS_15.png)

重啟後，看到這個畫面便大功告成!!
![](/assets/lineageOS_16.png)

開始登入Google帳號
![](/assets/lineageOS_17.png)

可以正常使用囉!
![](/assets/lineageOS_18.png)