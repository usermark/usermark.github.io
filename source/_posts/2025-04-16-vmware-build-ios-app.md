---
title: "[iOS] 從無到有用 VMWare 打包 Flutter iOS app"
date: 2025-04-16 11:15:38
tags:
- VMWare
- MacOS
- Flutter
---
手邊 MacBook 版本太舊不能支援新版 Xcode 和上架 APP，無奈只好用 VMWare 安裝 MacOS 15.1 來打包並上架。又遇到 Apple ID 無法在 VMWare 模擬的 MacOS 登入，回"發生未知的錯誤"，需要手動自簽 CSR 憑證上傳 後台，接著後面還有一連串行為才能成功打包並上架。

<!--more-->
# 安裝 VMware Workstation 17 Pro

首先要準備好 Sequoia.iso，取得方式之一是用 MacBook 下載與製作，教學看這篇 [【詳細教學】macOS Sequoia ISO 下載與製作](https://www.drbuho.com/zh-tw/how-to/macos-sequoia-iso)

[在VMWare Workstation 16安裝MacOS](https://hackmd.io/@enoladne/BJjgo8R6F)看這篇就夠了。

https://github.com/DrDonk/unlocker/releases/tag/v4.2.7

![](/assets/vmware_unlock.png)

![](/assets/vmware_unlock2.png)

![](/assets/vmware_unlock3.png)

# 安裝 Xcode 16.2

[如何手動快速下載不同版本的 Xcode](https://blog.poychang.net/manually-download-multiple-versions-of-xcode/)

# 自簽 CSR 憑證

[iOS App 上架流程圖文教學](https://franksios.medium.com/ios-app上架流程圖文教學-724636ddc78b)

![](/assets/vmware_cer.png)

改為永遠信任

![](/assets/vmware_cer2.png)

# 安裝 Homebrew 和 git

至 <https://github.com/Homebrew/brew/releases/tag/4.4.31> 抓取 pkg 檔做安裝。

執行下方指令安裝 git，後面抓取 Flutter SDK 會用到
```shell
brew install git
```

# 安裝 VS Code 和 Flutter 外掛

# 安裝 CocoPods

**參考資料**

1. [How to properly install Cocoapods(You don’t have write permissions for the /Library/Ruby/Gems/2.6.0 directory)](https://jideije-emeka.medium.com/how-to-properly-install-cocoapods-you-dont-have-write-permissions-for-the-library-ruby-gems-2-6-0-41ba58ee9f2b)