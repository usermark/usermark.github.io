---
layout: article
title: '[Git] 在SourceTree上使用SSH Key與Github連線'
date: 2022-03-03 13:33
tags:
- Git
- SourceTree
---
Github已廢除https輸入帳密的方式，需改用SSH Key連線。

由於SourceTree內無法設定SSH Key，是直接讀取系統預設的，只能透過指令操作。
<!--more-->
# 產生SSH Key

輸入以下指令產生金鑰，儲存位置預設/.ssh/id_rsa即可，且不設定密碼。
```shell
ssh-keygen -t rsa -b 4096 -C "your_email"
```

# Github新增SSH Key

輸入以下指令查看公鑰內容，並複製輸出內容到剪貼簿。
{% tabs mac_windows %}
<!-- tab mac -->
```shell
cat ~/.ssh/id_rsa.pub
```
<!-- endtab -->
<!-- tab windows -->
```shell
type C:\Users\XXX\.ssh\id_rsa.pub
```

或是使用[PuTTYgen](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)查看，選單「Conversions」>「Import key」
![](/assets/puttygen.png)
<!-- endtab -->
{% endtabs %}

登入Github網頁版，到「Settings」>「SSH and GPG keys」>「New SSH key」，貼上剛複製的內容。

# SourceTree設定

更換遠端伺服器倉庫網址，網址會從https改成git開頭。
```
git remote set-url origin git@github.com:user_name/project.git
```

輸入以下指令進行連線測試，第一次會詢問是否要繼續連線，輸入yes繼續。
```shell
ssh -T git@github.com
```
需做該步驟，若直接在SourceTree上點fetch同樣會跳出https輸入帳密的方式，但就算輸入也不會成功。

**參考資料**
1. [【Git】使用 SSH 金鑰與 GitHub 連線](https://cynthiachuang.github.io/Generating-a-Ssh-Key-and-Adding-It-to-the-Github/)
2. [Github SSH 連線設定，確保 Mac SourceTree 正常運作](https://dev.twsiyuan.com/2018/10/add-an-ssh-key-to-github.html)