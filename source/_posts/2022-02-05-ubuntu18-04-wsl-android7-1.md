---
layout: article
title: "[Android] Ubuntu18.04 (WSL)編譯Android7.1"
date: 2022-02-05 13:41
tags:
  - Android
  - AOSP
  - WSL
---

手把手 AOSP 編譯入門，使用 Windows 環境操作。

<!--more-->

# (可選) 移動 Ubuntu 至其他硬碟

Linux 子系統預設安裝在主硬碟裡，編譯 AOSP 需要至少 200G 的空間，可能會發生主硬碟空間不足的問題，如果不會可跳過此步驟。

## 1.安裝 choco

使用 PowerShell 輸入以下指令

```
Set-ExecutionPolicy Bypass -Scope Process -Force; iex ((New-Object System.Net.WebClient).DownloadString(‘https://chocolatey.org/install.ps1')
```

## 2.安裝 LxRunOffline

使用 PowerShell 輸入以下指令

```
choco install lxrunoffline
```

## 3.在另一個硬碟建資料夾並設置權限

在 D 槽建立 wsl 資料夾，接著使用 PowerShell 輸入以下指令，來查看你的使用者名稱，例如我的輸出為 usermark-pc\mike。

```
whoami
```

使用以下指令來開啟 D:\wsl 資料夾權限

```
icacls D:\wsl /grant "usermark-pc\mike:(OI)(CI)(F)"
```

## 4.使用 LxRunOffline 來移動 Linux 子系統

使用 PowerShell 輸入以下指令，來查看 Linux 子系統的名稱，例如我的輸出為 Ubuntu。

```
wsl -l
```

使用以下指令移動

```
lxrunoffline move -n Ubuntu -d D:\wsl\installed\Ubuntu
```

移動過程需要一些時間，完成後可用以下指令確認，輸出訊息會是 d:\wsl\installed\Ubuntu

```
lxrunoffline get-dir -n Ubuntu
```

# 設置編譯環境

## 1.安装 JDK

使用 bash 輸入以下指令，參考<https://source.android.com/setup/build/older-versions#for-ubuntu-15-04>

```
sudo apt-get update
sudo apt-get install openjdk-8-jdk
```

## 2.安裝工具包

使用 bash 輸入以下指令，參考<https://source.android.com/setup/build/initializing#installing-required-packages-ubuntu-1804>

```
sudo apt-get install git-core gnupg flex bison build-essential zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386 libncurses5 lib32ncurses5-dev x11proto-core-dev libx11-dev lib32z1-dev libgl1-mesa-dev libxml2-utils xsltproc unzip fontconfig
```

## 3.開啟大小寫敏感

使用 PowerShell 輸入以下指令

```
fsutil.exe file setCaseSensitiveInfo D:\wsl enable
```

# 下載 AOSP

## 1.安裝 Repo

參考<https://source.android.com/setup/develop#installing-repo>

```shell
mkdir ~/bin
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```

## 2.初始化 Repo

參考<https://source.android.com/setup/build/downloading#initializing-a-repo-client>

```shell
# 建立資料夾，名稱自定，這裡命名為android
mkdir android
cd android

# 設置git的使用者名稱和email
git config --global user.name $your_name
git config --global user.email $your_email
```

選定 AOSP 版本和分支，如下圖這裡選擇的是 android-7.1.0_r7，參考<https://source.android.com/setup/start/build-numbers#source-code-tags-and-builds>
![](/assets/android_version.png)
Repo 要用 python3 運行，但系統預設為 python2，有解法是採用以下方案，將 python 指令指向 python3 指令

```
sudo ln -s /usr/bin/python3 /usr/bin/python
```

repo sync 時可以，但編譯時還是需要用到 python2，就會出現該錯誤

```
SyntaxError: Missing parentheses in call to 'print'.
```

因此透過 python3 去呼叫 repo 較合適，系統預設則不去動。

sync 同步到本地，過程需要一些時間。

```
python3 ~/bin/repo init -u https://android.googlesource.com/platform/manifest -b android-7.1.0_r7
python3 ~/bin/repo sync
```

## 3.(可選) 清空輸出資料夾

清空 out 目錄

```
m clean
```

# 編譯 AOSP

## 1.初始化相關指令

_每次重開指令視窗都要先呼叫，否則會找不到指令 m 和 emulator。_

```
source build/envsetup.sh
```

## 2.選擇編譯目標

參考<https://source.android.com/setup/build/building#choose-a-target>

```
lunch aosp_arm-eng
```

## 3.讓 bash 支持 32 位元運行

安裝 qemu 並設置 binfmt

```shell
sudo apt install qemu-user-static
sudo update-binfmts --install i386 /usr/bin/qemu-i386-static --magic '\x7fELF\x01\x01\x01\x03\x00\x00\x00\x00\x00\x00\x00\x00\x03\x00\x03\x00\x01\x00\x00\x00' --mask '\xff\xff\xff\xff\xff\xff\xff\xfc\xff\xff\xff\xff\xff\xff\xff\xff\xf8\xff\xff\xff\xff\xff\xff\xff'
```

啟動服務

```
sudo service binfmt-support start
```

_每次重開指令視窗都要先呼叫，否則會出現以下錯誤。_

```
prebuilts/misc/linux-x86/bison/bison: cannot execute binary file: Exec format error
```

## 4.開始編譯

使用 m -jN 指令來編譯，此處的 N 一般建議設置為 cpu 核心數的 1-2 倍，參考<https://source.android.com/setup/build/building#build-the-code>

```
m -j8
```

在編譯期間，將防毒軟體關閉，可以增進檔案系統的效能。

## 5.錯誤處理

### 報錯: ftruncate(fd_out, GetSize()): Invalid argument

```
cd build/tools/ijar
```

修改 zip.cc，註解 935 行如下：

```c
int OutputZipFile::Finish() {
  if (fd_out > 0) {
    WriteCentralDirectory();
    // if (ftruncate(fd_out, GetSize()) < 0) {
    //   return error("ftruncate(fd_out, GetSize()): %s", strerror(errno));
    // }
    if (close(fd_out) < 0) {
      return error("close(fd_out): %s", strerror(errno));
    }
    fd_out = -1;
  }
  return 0;
}
```

保存並編譯產生 ijar

```
g++ -o ijar classfile.cc zip.cc ijar.cc -lz
```

強制替換 ijar

```
cp -f ijar ../../../out/host/linux-x86/bin/ijar
```

### 報錯: dex2oat did not finish after 2850 seconds

修改 build/core 資料夾下的文件，$(DEX2OAT)後加上-j1 參數

dex_preopt_libart.mk

```make
$(hide) ANDROID_LOG_TAGS="*:e" $(DEX2OAT) -j1 \
	--runtime-arg -Xms$(DEX2OAT_XMS) --runtime-arg -Xmx$(DEX2OAT_XMX) \
	--runtime-arg -classpath --runtime-arg $(DEX2OAT_CLASSPATH)
```

dex_preopt_libart_boot.mk

```make
$(hide) ANDROID_LOG_TAGS="*:e" $(DEX2OAT) -j1 --runtime-arg -Xms$(DEX2OAT_IMAGE_XMS) \
	--runtime-arg -Xmx$(DEX2OAT_IMAGE_XMX) \
	--image-classes=$(PRELOADED_CLASSES) \
```

### 報錯: SSL error when connecting to the Jack server. Try 'jack-diagnose'

修改/etc/java-8-openjdk/security/java.security，找到 jdk.tls.disabledAlgorithms，取消 TLSv1, TLSv1.1

```
jdk.tls.disabledAlgorithms=SSLv3, RC4, DES, MD5withRSA, \
    DH keySize < 1024, EC keySize < 224, 3DES_EDE_CBC, anon, NULL, \
    include jdk.disabled.namedCurves
```

重啟 jack 服務

```
./prebuilts/sdk/tools/jack-admin kill-server
./prebuilts/sdk/tools/jack-admin start-server
```

### 報錯: GC overhead limit exceeded. Try increasing heap size with java option '-Xmx<size>'.

運行 jack-server 編譯 AOSP 需要 3G 的記憶體配置，若電腦總記憶體小於 16G，會產生該錯誤，可使用以下指令查看可配置的記憶體大小。

```
java -XshowSettings 2>&1 | grep Heap
```

例如在總記憶體 8G 的電腦輸出結果為 1.77G，參考<https://2net.co.uk/blog/jack-server.html>

```shell
sudo vim ~/.bash_profile
# 最後一行加上
export JACK_SERVER_VM_ARGUMENTS="-Dfile.encoding=UTF-8 -XX:+TieredCompilation -Xmx4g"
# 儲存並離開後
source ~/.bash_profile
```

重啟 jack 服務

```
./prebuilts/sdk/tools/jack-admin kill-server
./prebuilts/sdk/tools/jack-admin start-server
```

# 運行模擬器

```
emulator
```

因為 wsl 為指令模式，沒視窗可顯示，會出現下列錯誤。

```
SDL init failure, reason is: No available video device
```

# (可選) 重置 AOSP 異動

參考<https://stackoverflow.com/questions/5012163/how-to-discard-changes-using-repo>

```shell
# 重置異動
python3 ~/bin/repo forall -vc "git reset --hard"
# 檢查異動
python3 ~/bin/repo status
```

**參考資料**

1. [Ubuntu18.04 (WSL) 编译 RK3399 Android8.1 源码](https://blog.csdn.net/tangjie134/article/details/102796330)
2. [Android FrameWork 学习（一）Android 7.0 系统源码下载\编译](https://www.jianshu.com/p/6af0bb7c1e70)
3. [完整指引如何編譯 AOSP (Build Android P)，整合 GMS 及刷機 (Pixel 2)](https://c55jeremy-tech.blogspot.com/2019/04/aosppixel-2-romrom.html)
4. [在 Windows 10 上運行 Ubuntu Linux 子系統並將系統移動至其他硬碟](https://medium.com/@chenwy0806/%E5%9C%A8-windows-10-%E4%B8%8A%E9%81%8B%E8%A1%8C-ubuntu-linux-%E5%AD%90%E7%B3%BB%E7%B5%B1%E4%B8%A6%E5%B0%87%E7%B3%BB%E7%B5%B1%E7%A7%BB%E5%8B%95%E8%87%B3%E5%85%B6%E4%BB%96%E7%A1%AC%E7%A2%9F-57e8faaa3e04)
