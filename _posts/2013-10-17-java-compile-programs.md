---
layout: post
title: '[Java] 編譯程式'
date: 2013-10-17 07:43
categories: 
---
假設 C:\demo 下有一個 HelloWorld.java，且無套件設定。
``` java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("Hello World!");
    }
}
```
javac [檔案來源]
java -cp [路徑] [完整的類別名稱]
```
C:\>javac \demo\HelloWorld.java
C:\>java -cp \demo HelloWorld
Hello World!
```
假設更改 HelloWorld.java 的套件設定為 io.github.usermark;
因此，這個類別名稱不再是 HelloWorld，而是 io.github.usermark.HelloWorld。
```
C:\>javac \demo\HelloWorld.java
C:\>java -cp \demo HelloWorld
Exception in thread "main" java.lang.NoClassDefFoundError: HelloWorld (wrong name: io/github/usermark/HelloWorld)
        at java.lang.ClassLoader.defineClass1(Native Method)
        at java.lang.ClassLoader.defineClass(Unknown Source)
        at java.security.SecureClassLoader.defineClass(Unknown Source)
        at java.net.URLClassLoader.defineClass(Unknown Source)
        at java.net.URLClassLoader.access$100(Unknown Source)
        at java.net.URLClassLoader$1.run(Unknown Source)
        at java.net.URLClassLoader$1.run(Unknown Source)
        at java.security.AccessController.doPrivileged(Native Method)
        at java.net.URLClassLoader.findClass(Unknown Source)
        at java.lang.ClassLoader.loadClass(Unknown Source)
        at sun.misc.Launcher$AppClassLoader.loadClass(Unknown Source)
        at java.lang.ClassLoader.loadClass(Unknown Source)
        at sun.launcher.LauncherHelper.checkAndLoadMain(Unknown Source)

C:\>java -cp \demo io.github.usermark.HelloWorld
錯誤: 找不到或無法載入主要類別 io.github.usermark.HelloWorld
```
解決方法是：移動 HelloWorld.java 至 C:\demo\io\github\usermark 目錄下。
```
C:\>java -cp \demo io.github.usermark.HelloWorld
Hello World!
```
另一個方法則是：在編譯時加上 -d 引數。

javac -d [路徑] [檔案來源]
```
C:\>javac -d \demo \demo\HelloWorld.java
C:\>java -cp \demo io.github.usermark.HelloWorld
Hello World!
```