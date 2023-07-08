---
layout: article
title: '[Java] 語言基礎'
date: 2013-10-27 01:15
tags: Java
---
<!--more-->
# 存取修飾元 (Modifier)

| | |
| :----------------- | :---------------------------------- |
| - private          | 同一個 class 才可存取               |
| ~ default 無修飾元  | 同一個 package 的 class 才可存取   |
| # protected        | 同一個 package 的 class 才可存取   |
|                    | 不同 package 的要有繼承關係才可存取 |
| + public           | 皆可存取                            |

# 基本資料型別 (Primitive Type)
二進位用來表示一個簡單的正負值 (true/false)，1個位元組 (byte) 代表8個位元 (bits)。
- 千位元組 (KB) = 2^10
- 兆位元組 (MB) = 2^20
- 吉位元組 (GB) = 2^30
- 位元組 (byte) 佔1位元組
- 短整數 (short) 佔2位元組
- 整數 (int) 佔4位元組
- 長整數 (long) 佔8位元組
- 浮點數 (float) 佔4位元組
- 雙精度浮點數 (double) 佔8位元組
- 字元 (char) 佔2位元組，Java 的字元採用 Unicode 編碼，所以一個中文字 (2 bytes) 與一個英文字母 (1 byte) 在 Java 中同樣都是用一個字元來表示。

## 字串
字串透過字元陣列來維護，建立字串後不能修改它的字元內容。
應避免用 + 串接字串，Java 會透過 StringBuilder (非同步) 或 StringBuffer (同步) 來產生新的字串。
``` java
String s1 = "Hello";
String s2 = "World";
String s = (new StringBuilder()).append(s1).append(s2).toString();
System.out.println(s);
```