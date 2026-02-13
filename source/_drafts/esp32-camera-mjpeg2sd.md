---
title: "[ESP32] 安信可 ESP32-CAM 鏡頭實作錄影和回放"
tags:
- ESP32
- IOT
- Servo
---
目的是有一個家用攝影機，隨時錄製，且可移動鏡頭，並在需要時回放觀看。
已有完整的專案<https://github.com/s60sc/ESP32-CAM_MJPEG2SD>，這篇記錄筆者一步步摸索踩坑的過程。
<!--more-->

# ESP32-CAM_MJPEG2SD

下載整個專案

修改 appGlobals.h，保留 #define CAMERA_MODEL_AI_THINKER，註解掉 #define CAMERA_MODEL_ESP32_S3_CAM

# 遠端遙控

3D 列印 Pan-Tilt 支架<https://www.thingiverse.com/thing:2467743>