---
layout: post
title: '[Java] 物件導向'
date: 2013-10-27 05:44
categories: 
description: toString() 自訂 Box 類別，覆寫 toString()，可以輸出有用的資訊，而不再是 Box@1fb8ee3。
---
# toString()
自訂 Box 類別，覆寫 toString()，可以輸出有用的資訊，而不再是 Box@1fb8ee3。
``` java
public class Box {
    private int width;
    private int height;
    
    public Box(int width, int height) {
        this.width = width;
        this.height = height;
    }
    
    @Override
    public String toString() {
        return "寬:" + width + "px, 高:" + height + "px";
    }
    
    public static void main(String[] args) {
        Box box = new Box(100, 50);
        System.out.println(box.toString()); 
    }
}
// ***結果***
// 寬:100px, 高:50px
```