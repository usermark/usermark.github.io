---
title: "[iOS] 從無到有用 VMWare 打包 Flutter iOS app"
date: 2025-04-16 11:15:38
tags:
- VMWare
- MacOS
- Flutter
---
手邊 MacBook 版本太舊不能支援新版 XCode 和上架 APP，無奈只好用 VMWare 安裝 MacOS 15.1 來打包並上架。又遇到 Apple ID 無法在 VMWare 模擬的 MacOS 登入，需要手動自簽 CSR 憑證上傳 後台，接著後面還有一連串行為才能成功打包並上架。

<!--more-->
# 安裝 VMWare

首先要準備好 Sequoia.iso，取得方式之一是用 MacBook 下載與製作，教學看這篇 [【詳細教學】macOS Sequoia ISO 下載與製作](https://www.drbuho.com/zh-tw/how-to/macos-sequoia-iso)

# 安裝 XCode

# 自簽 CSR 憑證

# 安裝 Homebrew 和 git

至 <https://github.com/Homebrew/brew/releases/tag/4.4.31> 抓取 pkg 檔做安裝。

執行下方指令安裝 git，後面抓取 Flutter SDK 會用到
```shell
brew install git
```

# 安裝 VS Code 和 Flutter 外掛