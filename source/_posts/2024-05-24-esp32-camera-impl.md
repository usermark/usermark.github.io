---
title: "[ESP32] 安信可ESP32-CAM鏡頭實作"
date: 2024-05-24 09:06:05
tags:
- ESP32
- IOT
---
使用官方提供的範例，不需要Linux環境，使用Windows就可以囉。
<!--more-->
首先下載官方範例，含底下的submodule
```bash
git clone --recurse-submodules https://github.com/Ai-Thinker-Open/Ai-Thinker-Open_ESP32-CAMERA_LAN.git
```

下載的submodule還蠻多的，若遇到timeout，可執行下方指令繼續
```bash
git submodule update --recursive
```

# 建立esp-idf編譯環境

下載完後，底下有個esp-idf資料夾，進去後執行
```bash
install.bat
```

```
All done! You can now run:
   export.bat
```
等看到這個輸出結果，就可以執行export.bat

```
Done! You can now compile ESP-IDF projects.
Go to the project directory and run:

  idf.py build
```

到這裡，esp-idf編譯環境已經建好囉！

# 範例

cd切換到 examples\single_chip\camera_web_server 資料夾後執行
```bash
idf.py menuconfig
```

Camera Web Server —>Camera Pins —> Select Camera Pinout —> 選擇 ESP32-CAM by AI-Thinker

```bash
idf.py build
```

# 編譯錯誤處理

## No such file or directory
```
../main/app_sd.c:15:10: fatal error: esp_vfs_fat.h: No such file or directory
 #include "esp_vfs_fat.h"
          ^~~~~~~~~~~~~~~

../main/app_smart_wifi.c:16:10: fatal error: esp_wpa2.h: No such file or directory
 #include "esp_wpa2.h"
          ^~~~~~~~~~~~
compilation terminated.
[855/860] Building C object esp-idf/main/CMakeFiles/__idf_main.dir/app_httpd.c.obj
ninja: build stopped: subcommand failed.
ninja failed with exit code 1
```
因缺少依賴包，至專案底下修改 examples\single_chip\camera_web_server\main\CMakeLists.txt
於COMPONENT_REQUIRES新增fatfs, vfs, wpa_supplicant
```make
set(COMPONENT_SRCS "app_main.c" "app_camera.c" "app_httpd.c" "app_mdns.c" "app_wifi.c" "app_sd.c" "app_uart.c""app_smart_wifi.c")
set(COMPONENT_ADD_INCLUDEDIRS "include")

set(COMPONENT_REQUIRES
    esp32-camera
    esp-face
    nvs_flash
    esp_http_server
    fb_gfx
    mdns
    fatfs
    vfs
    wpa_supplicant
    )

set(COMPONENT_EMBED_FILES
        "www/index_ov2640.html.gz"
        "www/index_ov3660.html.gz"
        "www/index_ov5640.html.gz"
        "www/monitor.html.gz")

register_component()
```

## undefined reference to `app_board_main'
```
[18/19] Linking CXX executable camera_web_server.elf
FAILED: camera_web_server.elf
cmd.exe /C "cd . && C:\Users\XXX\.espressif\tools\xtensa-esp32-elf\esp-2019r2-8.2.0\xtensa-esp32-elf\bin\xtensa-esp32-elf-g++.exe  -mlongcalls -Wno-frame-address  -nostdlib @CMakeFiles\camera_web_server.elf.rsp  -o camera_web_server.elf  && cd ."
c:/users/XXX/.espressif/tools/xtensa-esp32-elf/esp-2019r2-8.2.0/xtensa-esp32-elf/bin/../lib/gcc/xtensa-esp32-elf/8.2.0/../../../../xtensa-esp32-elf/bin/ld.exe: esp-idf/main/libmain.a(app_main.c.obj):(.literal.app_main+0x8): undefined reference to `app_board_main'
c:/users/XXX/.espressif/tools/xtensa-esp32-elf/esp-2019r2-8.2.0/xtensa-esp32-elf/bin/../lib/gcc/xtensa-esp32-elf/8.2.0/../../../../xtensa-esp32-elf/bin/ld.exe: esp-idf/main/libmain.a(app_main.c.obj): in function `app_main':
d:\ai-thinker-open_esp32-camera_lan\examples\single_chip\camera_web_server\build/../main/app_main.c:32: undefined reference to `app_board_main'
collect2.exe: error: ld returned 1 exit status
ninja: build stopped: subcommand failed.
ninja failed with exit code 1
```

修改 examples\single_chip\camera_web_server\main\CMakeLists.txt
於set(COMPONENT_SRCS 新增app_board.c
```make
set(COMPONENT_SRCS "app_main.c" "app_camera.c" "app_httpd.c" "app_mdns.c" "app_wifi.c" "app_sd.c" "app_uart.c""app_smart_wifi.c" "app_board.c")
```

# 編譯結果

```
Project build complete. To flash, run this command:
C:\Users\XXX\.espressif\python_env\idf4.0_py3.7_env\Scripts\python.exe ..\..\..\esp-idf\components\esptool_py\esptool\esptool.py -p (PORT) -b 460800 --before default_reset --after hard_reset write_flash --flash_mode dio --flash_size detect --flash_freq 80m 0x1000 build\bootloader\bootloader.bin 0x8000 build\partition_table\partition-table.bin 0x10000 build\camera_web_server.bin
or run 'idf.py -p (PORT) flash'
```

**參考資料**

1. [安信可 ESP32-CAM 摄像头开发 demo--局域网拍照、实时视频、人脸识别](https://aithinker.blog.csdn.net/article/details/108000974?ydreferer=aHR0cHM6Ly9kb2NzLmFpLXRoaW5rZXIuY29tLw%3D%3D)
