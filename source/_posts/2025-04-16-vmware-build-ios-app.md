---
title: "[iOS] 從無到有用 VMware 打包 Flutter iOS app"
date: 2025-04-16 11:15:38
tags:
- VMware
- MacOS
- Flutter
---
手邊 MacBook 版本太舊不能支援新版 Xcode 和上架 APP，無奈只好用 VMware 安裝 MacOS 15.1 來打包並上架。又遇到 Apple ID 無法在 VMware 模擬的 MacOS 登入，回"發生未知的錯誤"，需要手動自簽憑證請求上傳 Apple Developer 後台，接著後面還有一連串行為才能成功打包並上架。

<!--more-->
# 安裝 VMware Workstation 17 Pro

首先要準備好 Sequoia.iso，取得方式之一是用 MacBook 下載與製作，教學看這篇 [【詳細教學】macOS Sequoia ISO 下載與製作](https://www.drbuho.com/zh-tw/how-to/macos-sequoia-iso)。
然後下載 [unlocker](https://github.com/DrDonk/unlocker/releases/tag/v4.2.7)，再依[在VMware Workstation 16安裝MacOS](https://hackmd.io/@enoladne/BJjgo8R6F)的步驟裝好 VMware。
特別注意的是：裝好後畫面會小小的，不匹配視窗大小，需要進一步安裝 VMware Tools，找到剛 unlocker 底下的 iso 資料夾，將 darwin.iso 檔掛載到光碟機。

![](/assets/vmware_unlock.png)

開機後安裝完成，再去「隱私權與安全性」底下的「詳細資訊...」，把 VMware, Inc. 打開即可。

![](/assets/vmware_unlock2.png)

![](/assets/vmware_unlock3.png)

# 安裝 Xcode 16.2

至 https://developer.apple.com/downloads/more 搜尋 Xcode 16.2 來下載和安裝，該頁面需要登入 Apple ID。

# 自簽憑證請求

最關鍵的來了，原本若有使用 Apple ID 登入 Xcode 的，會自動在本機產生對應檔案，不用額外做設定。然而 VMware 模擬的 MacOS 無法登 Apple ID，會回”發生未知的錯誤”。因此 cer 憑證檔、Identifier、Provisioning Profile 都要手動處理。
主要參考這篇 [iOS App 上架流程圖文教學](https://franksios.medium.com/ios-app上架流程圖文教學-724636ddc78b)，底下則是筆者實作的。

## 產生 cer 憑證檔

在產生 cer 憑證檔之前必須先產生 certSigningRequest (CSR) 檔，開啟「鑰匙圖存取」，工具列打開「鑰匙圖存取」->「憑證輔助程式」->「從憑證授權要求憑證」。
1. 使用者電子郵件填入 Apple ID
2. 已將要求設為「儲存到硬碟」
3. 勾選「指定密鑰配對資訊」
4. 「密鑰大小」和「演算法」分別使用預設的 2048 bits 和 RSA

產出 CSR 檔後，至 Apple Developer 後台「Certificates」新增憑證，要選取「iOS Distribution（App Store Connect and Ad Hoc）」，並上傳前面產好的 CSR 檔。
![](/assets/apple_cer.png)

下載 cer 檔至本機後，記得點擊做安裝，安裝後會看到紅色不受信任
![](/assets/vmware_cer.png)

手動改為「永遠信任」即可

![](/assets/vmware_cer2.png)

## 產生 Identifier

跳過不贅述。

## 產生 Provisioning Profile

至 Apple Developer 後台「Profiles」新增，要選取「App Store Connect」，剩下依指示操作即可。

![](/assets/apple_profile.png)

# 安裝 Homebrew 和 git

至 <https://github.com/Homebrew/brew/releases/tag/4.4.31> 抓取 pkg 檔做安裝。
安裝完後執行指令確認
```shell
brew doctor
```

執行下方指令安裝 git，後面抓取 Flutter SDK 會用到
```shell
brew install git
```

# 安裝 VS Code 和 Flutter 外掛

跳過不贅述，但記得設定 PATH

```shell
echo export PATH=$HOME/development/flutter/bin:$PATH >> ~/.zshenv
```

# 安裝 CocoaPods

第二個挑戰在這。

依序輸入指令安裝最新版 ruby，需要跑一段時間。
```shell
brew install chruby ruby-install
ruby-install ruby
```

設定 shell
```shell
echo "source $(brew --prefix)/opt/chruby/share/chruby/chruby.sh" >> ~/.zshrc
echo "source $(brew --prefix)/opt/chruby/share/chruby/auto.sh" >> ~/.zshrc
echo "chruby ruby" >> ~/.zshrc
```

重啟終端機後，執行指令確認版本
```shell
ruby -v
```

最後安裝 CocoaPods
```shell
gem install cocoapods
```
# 打包 ipa 上傳

一般來說，Xcode 選擇 Product > Archive > Distribute App > App Store Connect 便可成功上傳 app，但在沒有登入 Apple ID 的情況就會出錯。

![](/assets/xcode_archive.png)

須改用 xcrun altool 的方式，首先 Product > Archive > Distribute App > Custom > App Store Connect > Export 先打包出 ipa。
至 Apple Developer 後台，找到「App Store Connect API」，建立存取權限為「App管理」的金鑰，

![](/assets/apple_store_api_key.png)

並將 p8 檔下載下來，放至 ~/private_keys 目錄底下。
執行下方指令上傳 ipa，其中 api_key 對應後台金鑰 ID，apiIssuer 對應後台 Issuer ID
```shell
xcrun altool --upload-app -f your_app.ipa -t ios --apiKey XXX --apiIssuer XXX --verbose
```

等待執行完畢，就可以到 Apple Developer 後台 TestFlight 找到剛上傳的版本囉！

**參考資料**

1. [如何手動快速下載不同版本的 Xcode](https://blog.poychang.net/manually-download-multiple-versions-of-xcode/)
2. [How to properly install Cocoapods(You don’t have write permissions for the /Library/Ruby/Gems/2.6.0 directory)](https://jideije-emeka.medium.com/how-to-properly-install-cocoapods-you-dont-have-write-permissions-for-the-library-ruby-gems-2-6-0-41ba58ee9f2b)