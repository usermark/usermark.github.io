---
title: '[3D列印] RC遙控車入門-螺絲、軸承、馬達、電子變速器(ESC)、鋰離子電池、平衡充電器、插頭、2.4Ghz遙控器'
tags:
- 3D列印
---
想來印進階點的玩具，正好找到Thingiverse有人分享[OpenRC F1 car - 1:10 RC Car](https://www.thingiverse.com/thing:1193309)，便因緣際會入門RC這塊領域。在找材料時，就順便整理相關基礎知識於此。
<!--more-->

# 螺絲

用一張圖解釋，M3x8即對應直徑3mm，長度8mm。
![](/assets/m3.png)

# 軸承

又稱培林，分成深溝滾珠軸承、微型軸承、薄壁軸承(67, 68, 69系列)、止推滾珠軸承、小徑滾珠軸承(MR128系列)。軸承的端蓋主要分為ZZ、2RS和OPEN三種類型。
![](/assets/bearing.png)

# 馬達

分成直流馬達與交流馬達。

## 直流馬達

又可分為有刷直流馬達與無刷直流馬達兩種。有刷直流馬達便是四驅車的馬達，無刷直流馬達則較進階，少了碳刷更耐用，但價格也較高。
常用日本Mabuchi(萬寶至)的編號系統來為馬達分級，如380馬達、540馬達，就是分別指直徑約28mm及36mm的馬達。540馬達適用1/10遙控車。
T數越大扭矩越大，速度越慢；T數越小扭矩越小，速度越快。舉例21T, 27T, 35T, 45T, 55T, 80T，21T適合跑平路，55T適合爬坡。
![](/assets/motor.png)

## 交流馬達

最常見的是伺服馬達(servo motor)，是一種能夠精確控制旋轉位置、速度和輸出力矩的馬達。其中360度的伺服馬達是連續旋轉的，使用Arduino IDE內建的example跑的話，不會在指定角度停下來，而是分成快慢、順向逆向旋轉。

# 電子變速器/電變/電子速度控制器(ESC)

應用電子變速器的領域通常為遙控模型或單晶片(微控制器)，由於微控制器(MCU)無法承受高電流，電子變速器透過電池提供的直流電將微控制器(MCU)發出來的數位訊號，通過脈波寬度調變(PWM)的轉換控制輸出電流來達到調節藉此控制馬達轉速。
![](/assets/esc.png)

## 去電池電路(BEC)

是ESC的一部份。BEC允許遙控模型只搭載一組電池(馬達電力來源為主)而不需要兩組(一組馬達電力另一組給遙控電路)。

# 鋰離子電池(Li-ion)/鋰聚合物電池(LiPo) 

為現行手機使用的電池，電壓為3.7V；而遙控模型常用的2S電壓為7.4V。S數代表著電池串連的數量，2S車時速約 36~42 KM/H。C數代表爆發力。
參考[電池介紹影片](https://youtu.be/FhCqE8k54fE)

# 平衡充電器

# 防爆袋

# 小田宮, 大田宮, T, XT60, XT90, TRX插頭

![](/assets/wire.png)

XT60的線規為12AWG，參考[AWG尺寸對照表](https://zh.wikipedia.org/zh-tw/美国线规#AWG尺寸对照表)，可耐電流為20A，接頭的額定電流為30A。
![](/assets/xt60.jpg)

舉例，選用的電池輸出端為XT60(母)，而ESC對接頭為大田宮(公)頭，會需要大田宮轉XT60(公)轉接線。

# 2.4Ghz遙控器

# 零件中英文對照表

|中文                   |英文                   |
|-----------------------|----------------------|
|螺絲 M3*8 沉頭/皿頭內六角|M3x8 Countersunk screw|
|螺絲 M3*8 盤頭/丸頭內六角|M3x8 Pan head screw   |
|螺帽 M3                 |M3 nuts               |
|軸承(培林) 6701         |6701 high speed bearing 12x18x4mm|
|全金屬伺服馬達/伺服機    |Metal Gear 9g Micro Servo (MG90S)|
|超旺540/35T有刷馬達     |Surpass Hobby 35T Brushed 540 size motor|
|60A有刷電子速度控制器    |60A Brushed ESC       |

**參考資料**
1. [螺絲規格](https://screwtechbuy.com/collections/%E8%9E%BA%E7%B5%B2-f%E9%A0%AD)
2. [軸承規格全解析：常見類型、尺寸、材質與應用考慮](https://iskbearing.com.tw/news/knowledge/introduction-to-bearing-specifications)
3. [馬達種類有哪些？用途、類型、挑選注意事項一次看！](https://www.sesamemotor.com/blog_detail/motor-select-tips)