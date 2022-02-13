---
layout: post
title: '[JavaScript] Closure'
date: 2013-10-27 01:05
categories: 
description: 可以用來模擬 Java 的 private 封裝特性。
---
可以用來模擬 Java 的 private 封裝特性。
``` js
function MyObject(name, size) {
    this._name = name;
    this._size = size;
    this.getSize = function() { // 這邊建立了一個 Closure
        return this._size;
    }
}
var myObject = new MyObject("Justin", 20);
alert(myObject.getSize());
```
每次用 new 建立 MyObject 實體時，都會建立一個新的函式物件，並讓 getSize() 參考，在程式中隱含建立一個 Closure，需注意資源綁定的問題。
``` js
function MyObject(name, size) {
    this._name = name;
    this._size = size;
}
MyObject.prototype.getSize = function() {
    return this._size;
}
```
可以避免產生 Closure，而要注意的是：prototype 可以在建構式到使用 new 呼叫之間修改。建構式被呼叫前，設定給 prototype 的東西才會被物件繼承。
``` js
MyObject.prototype.color = "red";
var obj1 = new MyObject(); // obj1.color 是 "red"

MyObject.prototype.color = "blue";
MyObject.prototype.sound = "Ah...";
var obj2 = new MyObject(); // obj2.color 是 "blue"，obj2.sound 是 "Ah..."
```