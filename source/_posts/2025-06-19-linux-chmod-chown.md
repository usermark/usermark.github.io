---
title: '[Linux] chmod chown 權限控制'
date: 2025-06-19 17:38:34
tags:
- Linux
---
剛接觸 Linux 相關系統的一定要會，可以少走很多冤望路。
<!--more-->

可輸入指令列出每個檔案擁有的權限
```bash
ls -la
```

# chmod

控制使用者對於檔案或目錄操作的權限

將 README.txt 權限設定成 使用者,群組,其它使用者 都能 RWX
```bash
chmod 777 README.txt
```

將 README.txt 權限設定成 使用者,群組 都能 RWX, 其它使用者 RW
```bash
chmod 775 README.txt
```

# chown

用來變更檔案或目錄的擁有者

修改 data 目錄的擁有者為 mike，群組為 www-data
```bash
chmod mike:www-data data
```