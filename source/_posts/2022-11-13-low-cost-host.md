---
layout: article
title: '[Web] 低成本架站'
date: 2022-11-13 04:36
tags: Nginx 小烏龜
---
家裡的網路環境是使用小烏龜，再加上ASUS無線AP。想讓外面可以用固定網域連進來，費了一番功夫，特別記錄下來。
<!--more-->

![](/assets/home_net.png)

第一步，先用Nginx架站 <http://localhost/>

![](/assets/nginx.png)

註冊一組 [no-ip](https://www.noip.com/)，並下載其工具，可將動態IP指派給固定網域，而且是免費的。

![](/assets/duc.png)

到這裡，直接使用固定網域是無法連上的，因為外部網路不知道要對應到內部網路的哪台主機上，接下來才是重點。
{:.info}

打開ASUS設定頁 <http://router.asus.com/>，登入後選擇 系統紀錄 > 系統資訊，移到最下方得知無線AP的IP位址 192.168.1.101。

![](/assets/asus_info.png)

選擇 外部網路 > 虛擬伺服器，啟用虛擬伺服器並輸入本機的IP位址 192.168.2.252，通訊埠為80。

![](/assets/asus_info2.png)

打開小烏龜的設定頁 <https://192.168.1.1>，切記要用https。選擇 Firewall 設定 > Port Forwarding，將無線AP的IP位址 192.168.1.101加上，大功告成！

![](/assets/port_forwarding.png)