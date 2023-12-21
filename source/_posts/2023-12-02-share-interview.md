---
layout: article
title: '[Java] 面試經驗分享'
date: 2023-12-02 21:59:59
tags: Java
---
最近面試幾間公司，沒想到久沒面試，發現很多東西要做好準備，才可增加錄取機會。
<!--more-->
以下皆屬個人經驗，僅供參考。
大部分都是先筆試再面試，若筆試完沒有直接面試，通常都是成績不好謝謝再聯絡。

# 面試前

準備好3分鐘的自我介紹，描述個人特質，工作經驗和做過的東西即可。要知道公司在做什麼，做足功課，大概知道有哪些產品也行。也可至[面試趣](https://interview.tw/)和[求職天眼通](https://www.qollie.com/)看看公司評價，通常負面訊息會較多，但有些要看部門，可評估未來有沒有機會遇到，且遇到後自己能不能夠接受。

# 筆試

筆試又細分成邏輯、專業、英文等。

## 邏輯題

邏輯試題大多都是英文，所以要有看懂英文的基本功，且又分成三種類型。一種是圖形推理，如下圖，會給前四張圖，要選出第五張圖會長怎樣，只要掌握好裡面物件分別是順時針或逆時針，或是數量增減即可。

![](/assets/graphical_reasoning.png)
<details>
    <summary>答案</summary>
B
</details>

另一種則是給一組數字或英文，找出規則後回答。例如：

a b c a b d a b e a b _
<details>
    <summary>答案</summary>
底線要填入答案是 f，因為 ab c ab d ab e 按此分群後，得知ab會固定出現，只是做分隔用，實際改變的依序是c d e，可以得出下個字母是 f。
</details>

第三種可先到 <https://yamol.tw/cat-邏輯推理-2101.htm> 準備，以下列出幾題自己遇到的。

Q: 今年某政府機關招考約僱員工，其錄取率 25%，錄取人數 310 人，請問報考人數為幾人？
(A) 1,290 人
(B) 1,240 人
(C) 1,170 人
(D) 1,150 人
<details>
    <summary>答案</summary>
B

310/0.25=1240
</details>

Q: 在計算軍隊的人數，已知人數介於 101-107 之間，每 3 人一數餘 1 人，每 5 人一數餘 1 人，每 7 人 一數餘 1 人，請問軍隊總人數為幾人？
(A) 106 人
(B) 104 人
(C) 103 人
(D) 101 人
<details>
    <summary>答案</summary>
A

3x5x7=105
105+1=106
</details>

Q: 總共有 50 個人回答問題，42 個人答對了 A 題，31 個人答對了 B 題，4 個人兩題都答錯。有多少人兩題都答對？
(A) 29 人
(B) 27 人
(C) 25 人
(D) 22 人
<details>
    <summary>答案</summary>
B

畫文式圖，答錯A題的有8個，答錯B題的有19個，都答錯的有4個。先計算出只要有答錯的為8+19-4=23，剩下便是兩題都答對的50-23=27。
</details>

Q: 甲、乙共有 3,600 元，當甲給乙 300 元，則甲、乙二人所得金額相同，請問甲有多少元？
(A) 1,500 元
(B) 2,100 元
(C) 2,400 元
(D) 2,700 元
<details>
    <summary>答案</summary>
B

甲-300=乙+300 => 甲=乙+600
代入前個公式 甲+乙=3600 => (乙+600)+乙=3600 => 2乙=3000 => 乙=1500
因此 甲=2100
</details>

Q: 有一數列為｢3, 15, 35, 63, 99, 143, A, …｣, 請問 A 的值為何？
(A) 195
(B) 210
(C) 240
(D) 270
<details>
    <summary>答案</summary>
A

1x3=3
3x5=15
5x7=35
7x9=63
9x11=99
11x13=143
13x15=195
</details>

Q: 小明與小華參加猜骰子點數的遊戲，如果猜中了可以得到 6 元，如果猜不中則要給莊家 1 元，不猜不算輸贏，只知道莊家總共丟了 20 次骰子，而且小明得到 50 元，小華得到 65 元，請問下列敘述何者 錯誤？
(A) 小明猜中 10 次、小華猜中 11 次
(B) 小明猜中 10 次、小華猜中 12 次
(C) 小明猜中 9 次、小華猜中 10 次
(D) 小明猜中 9 次、小華猜中 11 次
<details>
    <summary>答案</summary>
C

小明 50/6 必大於8
小華 65/6 必大於10，因此不可能只猜中10次
</details>

Q: 有一工作由甲獨做要 12 天，乙獨做要 24 天，丙獨做則要 48 天。今由甲、乙先合做一半，剩餘工作再由乙、丙合做，請問總共需時幾天？
(A) 8
(B) 10
(C) 12
(D) 16
<details>
    <summary>答案</summary>
C

假設工作量為X
甲一天工作量為X/12
乙一天工作量為X/24
丙一天工作量為X/48
因此甲乙一天工作量為X/12+X/24=X/8，完成一工作共需8天，一半的工作則為4天。
接著乙丙一天工作量為X/24+X/48=X/16，完成一工作共需16天，一半的工作則為8天。
4+8=12天
</details>

Q: 海賊甲、乙、丙三人參加宴會，會後被問到有哪些菜色。甲說：「有生魚片，也有魚翅，但沒有龍蝦」，乙答：「有生魚片，沒有魚翅，有龍蝦」，丙回：「沒有生魚片，有龍蝦」。若每人恰好都「對一項菜色說了謊話」，請問這個宴會有什麼菜？
(A) 有生魚片，沒有魚翅也沒有龍蝦
(B) 有生魚片和龍蝦，沒有魚翅
(C) 生魚片、魚翅與龍蝦都有
(D) 沒有生魚片，有魚翅和龍蝦
<details>
    <summary>答案</summary>
C
</details>

Q: 從甲地到乙地有陸路三條、水路二條路線可選擇；而從乙地到丙地有二條陸路、四條水路路線。請問某人走陸路從甲→乙→丙，再走水路從丙→乙→甲折返，請問共有多少種不同的走法？
(A) 14
(B) 30
(C) 36
(D) 48
<details>
    <summary>答案</summary>
D

甲地到乙地走陸路 3x2=6
乙地到丙地走水路 2x4=8
折返 6x8=48
</details>

Q: 有兩火車在平行的兩條軌道上相向對開，已知甲火車車身長為 96 公尺，行駛速度為 420 公尺／分鐘，乙火車車身長 84 公尺，行駛速度為 180 公尺／分鐘。請問兩火車從相遇到完全超過，共需要多少時間？
(A) 20 秒
(B) 18 秒
(C) 16 秒
(D) 14 秒
<details>
    <summary>答案</summary>
B

兩火車從相遇到完全超過，因此距離行駛距離為 96+84=180。甲乙行駛速度相加 420+180=600 公尺／分鐘，可轉換成 10 公尺／秒，180/10=18 秒。
</details>

Q: 甲、乙、丙、丁四人相約打桌球，預計打 5 個小時，每次 2 人上場 2 人休息，請問每人平均休息時間多久？
(A) 130 分鐘
(B) 150 分鐘
(C) 170 分鐘
(D) 190 分鐘
<details>
    <summary>答案</summary>
B

5小時等於300分鐘，假設甲乙先上場打150分鐘，丙丁休息，接著丙丁上場打150分鐘，換甲乙休息，平均休息時間為150分鐘。
</details>

Q: 甲、乙、丙、丁、戊五支球隊進行籃球比賽，每隊互賽一場進行循環賽。比賽的結果如下：甲隊 2 勝 2 敗； 乙隊 0 勝 4 敗；丙隊 1 勝 3 敗；丁隊 4 勝 0 敗。請問戊隊的成績如何？
(A) 4 勝 0 敗
(B) 3 勝 1 敗
(C) 2 勝 2 敗
(D) 1 勝 3 敗
<details>
    <summary>答案</summary>
B

甲乙丙丁勝場合計 2+0+1+4=7
甲乙丙丁敗場合計 2+4+3+0=9
加上戊後勝敗場合計要相同，得知戊至少有2勝，且勝場一定多於敗場2次，因此結果為 3 勝 1 敗。
</details>

Q: 如果我們不控制貨幣的供給，或是不打破石油輸出國家組織的力量，就無法控制通貨膨脹。請問由以上敘述，可推論出下列何者正確？
(A) 只要我們控制貨幣的供給，就能控制通貨膨脹
(B) 只要我們控制貨幣的供給，並且打破石油輸出國家組織的力量，就能控制通貨膨脹
(C) 如果我們能夠控制通貨膨脹，表示我們控制了貨幣的供給
(D) 如果我們打破石油輸出國家組織的力量，則我們控制了貨幣的供給
<details>
    <summary>答案</summary>
B
</details>

Q: 甲、乙、丙三人其中一個是麵包師傅、一個是計程車司機、一個是工程師。已知：A.甲和乙每天晚上一起下棋；B.乙和丙會一起去看棒球比賽；C.計程車司機喜歡收集硬幣，麵包師則喜歡集郵；D.計程車司機沒看過棒球比賽；E.丙不懂集郵有什麼樂趣。請問甲、乙、丙的職業依序為何？
(A) 計程車司機、麵包師傅、工程師
(B) 工程師、麵包師傅、計程車司機
(C) 麵包師傅、工程師、計程車司機
(D) 麵包師傅、計程車司機、工程師
<details>
    <summary>答案</summary>
A

乙和丙會一起去看棒球比賽，計程車司機沒看過棒球比賽，得知甲為計程車司機。
</details>

Q: 欲將 1,000 元大鈔換成 100 元、200 元、500 元的小鈔，每種鈔票皆可換可不換，請問兌換的方法有幾種？
(A) 10 種
(B) 9 種
(C) 8 種
(D) 7 種
<details>
    <summary>答案</summary>
A
</details>

## 專業題

### Java

Q: 撰寫方法反轉字串？
<details>
    <summary>答案</summary>
```java
public static String reverse(String s) {
    StringBuilder sb = new StringBuilder();
    for (int i = s.length() - 1; i >= 0; i--) {
        sb.append(s.chatAt(i));
    }
    return sb.toString();
}
```
</details>

Q: 將1000輸出成1,000，1000000000輸出成1,000,000,000
<details>
    <summary>答案</summary>
```java
DecimalFormat decimalFormat = new DecimalFormat("#,###,###");
String formattedAmount = decimalFormat.format(amount);
```
</details>

Q: 輸出以下內容
```
9*1=9
8*1=8 8*2=16
7*1=7 7*2=14 7*3=21
6*1=6 6*2=12 6*3=18 6*4=24
5*1=5 5*2=10 5*3=15 5*4=20 5*5=25
4*1=4 4*2=8 4*3=12 4*4=16 4*5=20 4*6=24
3*1=3 3*2=6 3*3=9 3*4=12 3*5=15 3*6=18 3*7=21
2*1=2 2*2=4 2*3=6 2*4=8 2*5=10 2*6=12 2*7=14 2*8=16
1*1=1 1*2=2 1*3=3 1*4=4 1*5=5 1*6=6 1*7=7 1*8=8 1*9=9
```
<details>
    <summary>答案</summary>
```java
for (int i = 9; i >= 1; i--) {
    for (int j = 1; j <= 10 - i; j++) {
        System.out.print(i + "*" + j + "=" + i*j + " ");
    }
    System.out.println();
}
```
</details>

Q: 解釋例外處理
<details>
    <summary>答案</summary>
分成執行時期例外，以及可修復的例外。使用可修復例外，要明確說明在程式碼中可能出問題的原因。
</details>

Q: 解釋interface, abstract class, extends
<details>
    <summary>答案</summary>
interface: 用於定義一組抽象方法，類別可實現多個interface達到多重繼承的效果。

abstract class: 不能被實體化，一個類別只可繼承一個抽象類別。
extends: 繼承允許取得並衍伸父類別的行為和定義。
</details>

Q: java中==和equals和hashCode的區別？
<details>
    <summary>答案</summary>
==用於比較記憶體位置，equals則用於比較內容，hashCode用來計算hash值，用於檢索。
</details>

### SQL

面試Java工程師的話，SQL幾乎是必考題，可至另一篇文章 [[SQL] 超經典SQL練習題](/2023/10/07/sql-50-quiz/) 做練習。

Q: 什麼是交易，rollback/commit分別代表什麼？
<details>
    <summary>答案</summary>
一個交易裡的運算動作應該是全部被執行，或完全不執行，滿足ACID的性質，分別代表單元性(Atomicity)、一致性(Consistency)、隔離性(Isolation)、永久性(Durability)。

commit用於將成功執行的交易結果永久保存於資料庫中。
rollback用於撤銷未執行成功的交易。
</details>

Q: 什麼是SQL injection？
<details>
    <summary>答案</summary>
是發生於應用程式與資料庫層的安全漏洞。是在輸入的字串之中夾帶SQL指令，在設計不良的程式當中忽略了字元檢查，那麼這些夾帶進去的惡意指令就會被資料庫伺服器誤認為是正常的SQL指令而執行，因此遭到破壞或是入侵。
</details>

Q: 如何避免
<details>
    <summary>答案</summary>
不把參數值寫死成字串，搭配資料型別判斷。
</details>

### 正則

<table>
<tr><td>.</td><td>任意字元</td></tr>
<tr><td>|</td><td>OR</td></tr>
<tr><td>\d</td><td>任意數字</td></tr>
<tr><td>\w</td><td>任意字</td></tr>
<tr><td>?</td><td>0或1個</td></tr>
<tr><td>*</td><td>0或多個</td></tr>
<tr><td>+</td><td>1或多個</td></tr>
<tr><td>^[A-Za-z0-9./*-]+$</td><td>排除特殊字元用</td></tr>
<tr><td>[A-Za-z]\d{9}</td><td>身份證</td></tr>
</table>

### Git

Q: Git從遠端儲存庫獲取並合併最新的分支更改的指令
<details>
    <summary>答案</summary>
```
git pull
```
</details>

Q: 解釋合併衝突
<details>
    <summary>答案</summary>
如果兩個或多個成員在兩個不同的分支 (即遠端和本地分支) 中對檔案的同一部分進行更改，Git 無法自動合併它們。
</details>

Q: 忽略檔案
<details>
    <summary>答案</summary>
```
echo <filename> >> .gitignore
```
</details>

### 網路

Q: 什麼是HTTP方法？
<details>
    <summary>答案</summary>
GET, POST, PUT, DELETE
</details>

Q: 使用REST服務的好處？
<details>
    <summary>答案</summary>
利用HTTP協定的普及，在客戶端和伺服器兩端，允許系統和設備可以不管其實作而輕鬆溝通。
</details>

Q: 解釋DES, AES, RSA
<details>
    <summary>答案</summary>
對稱加密算法，加解密使用同一個密鑰。包含DES, AES。

非對稱加密算法，加解密使用不同個密鑰，通常有公鑰和私鑰。包含RSA。
</details>

### 建構工具

Q: 說明Ant, Maven, Gradle其中一個
<details>
    <summary>答案</summary>
Maven是一個包羅萬象的建構工具。它是用來編譯、測試，以及部署Java專案。
</details>

### Android

Q: 何謂MVC？
<details>
    <summary>答案</summary>
依關注點分離，分成Model, View, Controller。

Model負責處理與資料相關的事，包含取得和儲存。
View負責畫面呈現。
Controller作為View和Model的橋樑，接收來自View的事件，去操控Model的資料。
</details>

Q: Android的四大組件是什麼？
<details>
    <summary>答案</summary>
Activity: 呈現使用介面。

Service: 長時間或長期在背景執行的服務。
ContentProvider: 儲存元件間共享的資料，可透過API取得。 
BroadcastReceiver: 接收系統或內部廣播的intent，像是顯示通知、啟動另一個元件。
</details>

Q: 簡述Handler、Message、Looper、MessageQueue？

### Python

電路板相關產業會考簡單的Python。

Q: 計算1到n加總
<details>
    <summary>答案</summary>
```python
sum = 0
for i in range(1, n + 1):
    sum += i
print(sum)
```
</details>

Q: 9x9乘法表
<details>
    <summary>答案</summary>
```python
for i in range(1, 10):
    for j in range(1, 10):
        print('%d*%d=%d' % (i, j, i*j), end=' ')
    print()
```
</details>

Q: 繪製等腰三角形
<details>
    <summary>答案</summary>
```python
for i in range(3):
    for j in range(3 - i - 1):
        print(' ', end='')
    for k in range(i * 2 + 1):
        print('*', end='')
    print()
```
</details>

Q: 找出最大值和最小值
l = [20, 10, 3, 4, 90]
<details>
    <summary>答案</summary>
```python
print(min(l))
print(max(l))
```
</details>

Q: 找出兩陣列重複元素，並依大小排序
a = {1, 3, 5}
b = {3, 2, 1}
<details>
    <summary>答案</summary>
```python
print(sorted(a & b))
```
</details>

### C#

Q: Override和Overload差別？
<details>
    <summary>答案</summary>
覆寫(Override)是指子類別可以覆寫父類別的方法內容，使該方法擁有不同於父類別的行為。
多載(Overload)指在一個類別中，定義多個名稱相同，但參數不同的方法。
</details>

Q: NuGet是什麼？
<details>
    <summary>答案</summary>
作為Visual Studio擴展，能夠簡化在Visual Studio項目中添加、更新和刪除庫（部署為程序包）的操作。
</details>

## 英文題

Q: I _____ say that to her if I were you.
(A) will not
(B) can not
(C) would not
(D) should not
<details>
    <summary>答案</summary>
C
</details>

Q: Today is a special day for us. _____ it’s our wedding anniversary.
(A) Apparently
(B) However
(C) Undoubtedly
(D) Actually
<details>
    <summary>答案</summary>
D
</details>

Q: Cellphones have a great ________ on our lives. Nowadays almost everyone uses a cellphone to communicate with others.
(A) trouble
(B) supply
(C) quality
(D) impact
<details>
    <summary>答案</summary>
D
</details>

# 面試

Q: 為何離開前公司？
超級重要必考題，[面試離職原因 : 為何離開前公司？我該誠實回答嗎[離職原因範例]](https://glints.com/tw/blog/離職原因-為何離開前公司，怎麼回答才算好？/) 直接看這篇文章就好，寫得很棒。

Q: 上一份工作的薪水是多少？
誠實為上策，不要浮報。且前工作薪水涉及個資，求職者並不一定要回答。

Q: 您期望的待遇薪資是？
傳產或科技業底薪不高，都是靠年終加上去的，所以若是找這類型的工作要談年薪。

Q: 主管交辦職務相關的工作，但對你來說非常困難，可以完成的機率不高，你會怎麼做？

# 結語

面試完，一定想知道有沒有機會錄取，好做下一步打算。可看面試官有沒有問這些問題：
* 最快何時可以來上班？
* 你還有面試其它家公司嗎？工作會找到時麼時候？—表示人資在評估你搶手的程度，成功報告的機會有多高。
* 您期望的待遇薪資是？—不太準，傳產或科技業都只是行政作業，必填欄位而已。

最後祝各位求職順利，未錄取也別氣餒，公司有各種考量，詳細可看這篇 [面試明明相談甚歡，為什麼沒被錄取？資深HR揭真實原因](https://blog.104.com.tw/interview-result-backfire-why-not-me/)，找到雙方都能合作愉快的，才能做得長久。

**參考資料**
1. [TQC+ Python 證照題目解答- JB 程式筆記](https://jbprogramnotes.com/2020/05/tqc-python-證照-題目解答/)