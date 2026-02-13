---
title: "[Raspberry Pi] 從遙控電燈開關開始 IOT 冒險 - 整合 Home Assistant"
tags:
  - Raspberry Pi
  - IOT
  - Home Assistant
  - 3D 列印
  - Servo
date: 2026-01-27 21:47:44
---
先從小專案開始，慢慢加大規模。
<!--more-->

# 遙控電燈開關

<table>
  <tr>
    <td><img src="/assets/light_switch.jpg"/></td>
    <td><img src="/assets/light_switch2.jpg"/></td>
  </tr>
</table>

3D 列印設計好的外殼 <https://www.tinkercad.com/things/2Xyr9GqkHPA-remote-control-light-switch>
使用 [ESP8266WebServer](/assets/ESP8266WebServer.py) 模組提供網站功能，以及 [Servo](/assets/servo.py) 模組控制伺服馬達的角度。

main.py
```python
from machine import Pin
from servo import Servo
import ESP8266WebServer
import network
import time

motor = Servo(pin=0)
angle = 0

def handleCmd(socket, args):
    global angle
    print(angle)
    if angle == 0:
        motor.move(50)
        angle = 50
    else:
        motor.move(0)
        angle = 0
    ESP8266WebServer.ok(socket, "200", "OK")

sta_if = network.WLAN(network.STA_IF)
sta_if.active(True)
sta_if.connect('無線網路名稱', '密碼')
while not sta_if.isconnected():
    print('連線中')
    time.sleep(1)
    pass
ESP8266WebServer.begin(80)
ESP8266WebServer.onPath("/", handleCmd)
print("address:" + sta_if.ifconfig()[0])

while True:
    ESP8266WebServer.handleClient()
```

Android 手機可以安裝 HTTP Request Shortcuts，方便用手機一鍵控制開關。

<iframe width="560" height="315" src="https://www.youtube.com/embed/UQaciRk0dsg?si=Yivd59a6kDZQ43jw" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

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

至「設定」>「附加元件」>「附加元件商店」，找到 File editor 並安裝，安裝後按下「啟動」，並「新增至側邊欄」

![](/assets/file_editor_config.png)

透過「File editor」，在 configuration.yaml 加入
```
rest_command:
  light_switch:
    url: "http://192.168.1.107/"
    method: GET
```
注意：記得先去路由器固定 ip，否則下次重新連線，會因為 ip 改變，而找不到。

至「開發工具」重新載入 YAML 設定後，就可以在「動作」搜尋到建立好的 light_switch，可按下「執行動作」確認服務正常。
![](/assets/ha_action.png)

# 新增電燈開關的卡片

至「總覽」>「編輯儀表板」>「接管」，接管後下次就不會再出現
![](/assets/ha_edit.png)

按下「自行編輯」
![](/assets/ha_edit2.png)

選「增加面板」
![](/assets/ha_edit3.png)

找到「按鈕面板」
![](/assets/ha_edit4.png)

往下找到「互動」並設定「圖示」和「執行動作」
![](/assets/ha_action_card.png)

或是直接「使用文字編輯器」
```
show_name: true
show_icon: true
type: button
tap_action:
  action: perform-action
  perform_action: rest_command.light_switch
  target: {}
icon: mdi:lightbulb-on
```

按下「儲存」並「完成」，就可以實際點擊玩看看。

<iframe width="560" height="315" src="https://www.youtube.com/embed/5A0sYedumG0?si=2Lm0814umF5Ssuxl" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

# 心得

實際玩過，發現有時會想直接手動開關電燈，但伺服馬達的舵片會卡著。初步想到的解法是舵片不要鎖，方便隨時可取下，不影響到手動去開關。