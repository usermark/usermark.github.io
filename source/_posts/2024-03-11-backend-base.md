---
title: '後端基礎-專有名詞'
date: 2024-03-11 15:17:57
tags:
---
接觸後端開發後，發現基礎沒打好，根本是新的世界，一些專有名詞都不懂，查完後彙整於此。
<!--more-->
# TLS

Netscape 開發了名為安全通訊端層（Secure Socket Layer，SSL）的上一代加密協議，TLS 由此演變而來。

參考 [什麼是 TLS（傳輸層安全性）？](https://www.cloudflare.com/zh-tw/learning/ssl/transport-layer-security-tls/)

# Cipher suite

使用在 TLS 時的演算法集合名稱。

參考 [簡介 HTTPS / TLS 安全通訊協議](https://rickhw.github.io/2021/08/20/ComputerScience/HTTPS-TLS/)

# GPG

GPG (The GNU Privacy Guard，又稱 GnuPG)，是一套實作 OpenPGP 標準規開源軟體。Github 也支援 GPG 金鑰。

參考 [GPG 筆記 - 產生 ECC 金鑰及加解密檔案](https://blog.darkthread.net/blog/gpg-gen-key-n-encrypt/)

# Docker

Docker 容器透過封裝應用程式與其相依性配置形成輕量的映像，且可直接運行在宿主機（host machine）上，減少對宿主機負荷的同時提供更多容器服務使用。而輕量化帶來的好處除了上述所提，它也讓 Docker 容器提供一致性的發布環境，使開發團隊可輕易地攜帶與轉換運行環境，提高協同作業的便利性。
從這張圖得知，Container 並沒有 OS，容量自然就小，而且啟動速度快。
![](/assets/VM_vs_Containers.png)
Volume 是 Docker 最推薦存放 persisting data（ 數據 ）的機制，因為他不會增加 container 的容量。

參考 [Docker 基本教學 - 從無到有 Docker-Beginners-Guide](https://github.com/twtrubiks/docker-tutorial)

# Kubernetes (K8s)

 是一項 Docker 容器編排工具。K8s 的前身是 Google 內部的 Borg 叢集管理系統，解決了過去管理者需各別管理不同主機內的容器，且為了確保所有容器服務正常，需設置對應的監控系統並手動重新部署有問題環境的困擾。
參考 [Kubernetes 是什麼？Docker、K8s、GKE 架構與優勢比較](https://blog.cloud-ace.tw/application-modernization/devops/docker-k8s-gke-intro/)

## Google Kubernetes Engine（GKE）

# Serverless (無伺服器)服務

使用 Severless 佈署，開發者可以完全不必考慮基礎架構的維護和擴展，只需專注於撰寫應用程式。

## Google Cloud Run

用於託管 Web 應用程式，包含網站或REST API 後端。
缺點：只能開放單一port。

參考 [什麼是 Cloud Run？Cloud Run 與 Cloud Functions 有哪些差別？關於 Cloud Run 的四大常見疑問解惑](https://www.microfusion.cloud/news/cloud-run-overview/)
實作參考 [【GCP】將 FastAPI 佈署上 Cloud Run](https://nijialin.com/2023/03/19/gcp-why-need-cloudrun-as-serverless/)