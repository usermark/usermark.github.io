---
title: "ZIP解壓縮密碼破解"
date: 2024-06-26 14:41:30
tags:
---

以下是參照[John the Ripper password cracker](https://www.openwall.com/john/)內的 doc\INSTALL-WINDOWS.txt，將操作步驟與實際遇到的記錄下來。

<!--more-->

# 安裝 John the Ripper password cracker

首先安裝[CygWin](https://cygwin.com/install.html)後，輸入以下指令安裝套件

```bash
$ setup-x86_64.exe -q -P make
$ setup-x86_64.exe -q -P libssl-devel -P libbz2-devel
$ setup-x86_64.exe -q -P libgmp-devel -P zlib-devel
$ setup-x86_64.exe -q -P libOpenCL-devel -P libcrypt-devel
```

抓取最新的 https://github.com/magnumripper/JohnTheRipper/archive/bleeding-jumbo.zip 並解壓縮。
開啟 CygWin terminal，並切換到 john-bleeding-jumbo\src 目錄，假設位於 C:\Users\XXX\Downloads\john-bleeding-jumbo\src，指令為

```bash
cd /cygdrive/c/Users/XXX/Downloads/john-bleeding-jumbo/src
```

進行編譯，需要一段時間

```bash
$ ./configure && make -s clean && make -sj4
$ make windows-package
```

編譯完成後，可以輸入下方指令測試下

```bash
$ cd ../run
$ john --test=0
```

# 測試

手動製作 demo.txt，並且壓縮成 demo.zip，加密方法為 AES-256，密碼為 passw0rd
或是拿手邊其它已有加密的 zip 檔做測試。
輸入下方指令取得 John 可以處理的格式

```bash
$ ./zip2john /cygdrive/c/Users/XXX/Downloads/demo.zip
demo.zip/demo.txt:$zip2$*0*3*0*5fc1daf064540d695750f1a159f5d8e7*7c06*b*183a8481561fccfb41176c*b331ede7817195ecf59d*$/zip2$:demo.txt:demo.zip:/cygdrive/c/Users/XXX/Downloads/demo.zip
```

只需要$夾著的這段

```
$zip2$*0*3*0*5fc1daf064540d695750f1a159f5d8e7*7c06*b*183a8481561fccfb41176c*b331ede7817195ecf59d*$/zip2$
```

存成檔案 hash.txt

# 安裝 hashcat

至 https://hashcat.net/hashcat/ 下載並解壓縮。並切換到該目錄

```bash
> hashcat -m 13600 -a 0 hash.txt example.dict --show
$zip2$*0*3*0*5fc1daf064540d695750f1a159f5d8e7*7c06*b*183a8481561fccfb41176c*b331ede7817195ecf59d*$/zip2$:passw0rd
```
![](/assets/zip2.png)

-m 指定雜湊方法的類型，可至<https://hashcat.net/wiki/doku.php?id=example_hashes>查看，使用$zip2$查到對應值為 13600
-a 破解模式: 0 字典, 1 組合, 3 暴力
-o 輸出檔案，也可以用 -\-show 直接顯示結果

其輸出結果最後面便是密碼囉!
![](/assets/hashcat.png)

這是使用字典破解，當然若是密碼更複雜，就要改用暴力破解，假設知道密碼為長度 6 的英文組合

```bash
hashcat -m 13600 -a 3 hash.txt ?l?l?l?l?l?l --show
```

注意，如果設定的範圍太大會出現錯誤
Integer overflow detected in keyspace of mask

規則可輸入 hashcat -h 查看

```
  ? | Charset
 ===+=========
  l | abcdefghijklmnopqrstuvwxyz [a-z]
  u | ABCDEFGHIJKLMNOPQRSTUVWXYZ [A-Z]
  d | 0123456789                 [0-9]
  h | 0123456789abcdef           [0-9a-f]
  H | 0123456789ABCDEF           [0-9A-F]
  s |  !"#$%&'()*+,-./:;<=>?@[\]^_`{|}~
  a | ?l?u?d?s
  b | 0x00 - 0xff
```
