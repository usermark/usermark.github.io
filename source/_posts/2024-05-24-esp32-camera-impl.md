---
title: "[ESP32] 安信可 ESP32-CAM 鏡頭實作"
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

# 編譯範例

cd切換到 examples\single_chip\camera_web_server 資料夾後執行
```bash
idf.py menuconfig
```

![](/assets/idf.png)

阿怎麼會是亂碼呢？
因為Python不支援中文編碼，輸入下方指令切換成英文編碼模式即可。
```bash
chcp 437
```

Camera Web Server ---> Camera Pins ---> Select Camera Pinout ---> 選擇「ESP32-CAM by AI-Thinker」
![](/assets/idf_fix.png)

![](/assets/idf_2.png)

![](/assets/idf_3.png)

![](/assets/idf_4.png)

設定好後，就可以開始編譯
```bash
idf.py build
```

# 編譯錯誤處理

編譯過程中會有錯誤要處理

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

## ccache: error: Failed to create temporary file

```
ccache: error: Failed to create temporary file for esp-idf/libsodium/CMakeFiles/__idf_libsodium.dir/libsodium/src/libsodium/crypto_box/curve25519xsalsa20poly1305/box_curve25519xsalsa20poly1305.c.obj: No such file or directory
[676/861] Building C object esp-idf/libsodium/CMakeFiles/_...bsodium/crypto_auth/hmacsha512256/auth_hmacsha512256.c.obj
ninja: build stopped: subcommand failed.
ninja failed with exit code 1
```

若有clean或者重新設定menuconfig，就會出現上面的問題
參考 [关于esp-idf编译时ccache错误导致在libsodium库报poly1305.c.obj类文件找不到的问题](https://blog.csdn.net/zhangjingxun12/article/details/117095349)，關閉ccache
```bash
idf.py --no-ccache build
```
還是不行的話，直接刪除 C:\Users\XXX\AppData\Roaming\\.ccache 資料夾。

```
Project build complete. To flash, run this command:
C:\Users\XXX\.espressif\python_env\idf4.0_py3.7_env\Scripts\python.exe ..\..\..\esp-idf\components\esptool_py\esptool\esptool.py -p (PORT) -b 460800 --before default_reset --after hard_reset write_flash --flash_mode dio --flash_size detect --flash_freq 80m 0x1000 build\bootloader\bootloader.bin 0x8000 build\partition_table\partition-table.bin 0x10000 build\camera_web_server.bin
or run 'idf.py -p (PORT) flash'
```

直到出現這幾行表示編譯成功，就可以開始接上ESP32-CAM做燒錄。

# 燒錄

![](/assets/esp32cam.jpg)

先至裝置管理員查看連接埠，設備對接口為COM6
![](/assets/com_search.png)

輸入指令做燒錄
```bash
idf.py -p COM6 flash
```

當初出現下方錯誤時
```
Executing "C:\Users\XXX\.espressif\python_env\idf4.0_py3.9_env\Scripts\python.exe D:\Ai-Thinker-Open_ESP32-CAMERA_LAN\esp-idf\components/esptool_py/esptool/esptool.py -p COM6 -b 460800 --before default_reset --after hard_reset write_flash @flash_project_args"...
esptool.py -p COM6 -b 460800 --before default_reset --after hard_reset write_flash --flash_mode dio --flash_freq 40m --flash_size 4MB 0x8000 partition_table/partition-table.bin 0x1000 bootloader/bootloader.bin 0x10000 camera_web_server.bin
usage: esptool write_flash [-h] [--erase-all] [--flash_freq {keep,40m,26m,20m,80m}]
                           [--flash_mode {keep,qio,qout,dio,dout}] [--flash_size FLASH_SIZE]
                           [--spi-connection SPI_CONNECTION] [--no-progress] [--verify] [--encrypt]
                           [--ignore-flash-encryption-efuse-setting] [--compress | --no-compress]
                           <address> <filename> [<address> <filename> ...]
esptool write_flash: error: argument <address> <filename>: Detected overlap at address: 0x8000 for file: partition_table/partition-table.bin
esptool.py failed with exit code 2
```

是因為bootloader.bin太大。
上述錯誤主要看這裡，bootloader.bin所能使用的空間為0x1000到0x8000，空間大小為28,672
```
0x8000 partition_table/partition-table.bin 
0x1000 bootloader/bootloader.bin 
0x10000 camera_web_server.bin
```
但實際查看大小為29,296
![](/assets/bootloader_bin.png)

調整log層級即可，先重新執行idf.py menuconfig
Bootloader config ---> Bootloader log verbosity ---> 選擇「Error」
![](/assets/idf_5.png)

重新編譯後，bootloader.bin大小剩下25,488
![](/assets/bootloader_bin_2.png)

再次執行idf.py -p COM6 flash，等出現這幾行表示燒錄成功。
```
Leaving...
Hard resetting via RTS pin...
Done
```

# 測試

輸入指令做監控
```bash
idf.py monitor -p COM6
```

若發現log一直重複出現，並且有看到這行
```
Brownout detector was triggered
```
表示供電不足，參考 [解決ESP32 CAM Brownout detector 問題](https://www.nmking.io/index.php/2022/12/15/713/)，所以如果中間有接USB延長線的先拿掉。
另外我的解法是調整Flash SPI speed，先重新執行idf.py menuconfig
Serial flasher config ---> Flash SPI speed ---> 選擇「40 MHz」
![](/assets/idf_6.png)

重新編譯和燒錄後，再次執行idf.py monitor
```
I (2921) camera wifi: wifi_init_softap finished.SSID:AiThinker password:12345678
I (3081) phy: phy_version: 4180, cb3948e, Sep 12 2019, 16:39:13, 0, 2
W (3081) phy_init: saving new calibration data because of checksum failure, mode(0)
I (3141) wifi:mode : softAP (XX:XX:XX:XX:XX:XX)
I (3141) wifi:Total power save buffer number: 8
I (3141) wifi:Init max length of beacon: 752/752
I (3141) wifi:Init max length of beacon: 752/752
I (3151) wifi:Set ps type: 0

I (3151) uart: queue free spaces: 20
I (3151) camera_httpd: Starting web server on port: '80'
I (3161) camera_httpd: Starting stream server on port: '81'
I (3171) system_api: Base MAC address is not set, read default base MAC address from BLK0 of EFUSE
I (3191) base MAC address: XX:XX:XX:XX:XX:XX
I (3191) esp-cam Version:
```
等出現這幾行，就可以連上http://192.168.4.1
![](/assets/cam_view.jpg)

實測解析度為VGA(640x480)時，亮度較夠，且比較順。

**參考資料**

1. [安信可 ESP32-CAM 摄像头开发 demo--局域网拍照、实时视频、人脸识别](https://aithinker.blog.csdn.net/article/details/108000974?ydreferer=aHR0cHM6Ly9kb2NzLmFpLXRoaW5rZXIuY29tLw%3D%3D)
2. [ESP32 idf.py menuconfig 乱码](https://blog.csdn.net/sudaroot/article/details/105215080)
