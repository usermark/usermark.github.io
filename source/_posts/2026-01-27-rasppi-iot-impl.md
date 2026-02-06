---
title: "[Raspberry Pi] 從遙控電燈開關開始 IOT 冒險"
tags:
  - Raspberry Pi
  - IOT
  - Home Assistant
date: 2026-01-27 21:47:44
---
先從小專案開始，慢慢加大規模。
<!--more-->

# 遙控電燈開關

使用[ESP8266WebServer](/assets/ESP8266WebServer.py)模組提供網站功能，以及Servo模組控制伺服馬達的角度。

```python

```

Android 手機可以安裝 HTTP Request Shortcuts，方便用手機一鍵控制開關。

# Home Assistant

使用樹莓派官方提供的 Raspberry Pi Imager 做燒錄。

![](/assets/rpi_imager_ha1.png)

![](/assets/rpi_imager_ha2.png)

![](/assets/rpi_imager_ha3.png)

![](/assets/rpi_imager_ha4.png)

![](/assets/rpi_imager_ha5.png)

![](/assets/rpi_imager_ha6.png)

燒錄完成後，插入 SD 卡並將樹莓派接上實體網路線後開啟。
等待一段時間便可用瀏覽器打開 http://homeassistant.local:8123/

![](/assets/ha1.png)

要特別講下，如何關機，筆者找了一下，所以特別紀錄和提醒。
先選「設定」，關機藏在右上角3個點，選「重新啟動 Home Assistant」後，要再按下「進階設定」，才會跳出「關閉系統」的選項。

![](/assets/ha2.png)

# 安裝 File editor

至附加元件商店，找到 File editor 並安裝，安裝後按下「啟動」

![](/assets/file_editor_config.png)

透過「File editor」，在 configuration.yaml 加入
```
rest_command:
  light_switch:
    url: "http://192.168.1.107/"
    method: GET
```
