---
layout: article
title: '[Eclipse] Add Gradle Nature無作用'
date: 2023-12-23 14:46:38
tags:
- Eclipse
- Gradle
---
正常要能作用才對，但我就是遇到了，把解法整理在這，給同樣踩到坑的夥伴。
<!--more-->
一般從Eclipse IDE這裡加上Gradle Nature後，就能手動新增build.gradle開始作業。
![](/assets/eclipse_add_gradle.png)

既然沒有作用的話，就自己手動建立起來。
首先至[Gradle官網](https://gradle.org/releases/)抓取最新的版本v8.5，解壓縮後，設定環境變數，例如
```
PATH D:\gradle-8.5\bin;%PATH%
```
接著輸入gradle指令看看是否設定成功。
下一步建立一個新的資料夾DemoGradle，輸入gradle init指令建立專案。
```bat
>mkdir DemoGradle

>cd DemoGradle

>gradle init

Select type of project to generate:
  1: basic
  2: application
  3: library
  4: Gradle plugin
Enter selection (default: basic) [1..4] 1

Select build script DSL:
  1: Kotlin
  2: Groovy
Enter selection (default: Kotlin) [1..2] 2

Project name (default: DemoGradle):
Generate build using new APIs and behavior (some features may change in the next minor release)? (default: no) [yes, no]


> Task :init
To learn more about Gradle by exploring our Samples at https://docs.gradle.org/8.5/samples

BUILD SUCCESSFUL in 12s
2 actionable tasks: 2 executed
```

進去查看會長這樣，把這幾個檔案選取後，複製進原先的專案。
![](/assets/gradle_folder.png)

切換回Eclipse IDE，打開專案的「Properties」，至「Project Natures」手動新增「Gradle Project Nature」。
![](/assets/manual_add_gradle.png)

接著刷新下Gradle Tasks，就會看到可執行的task囉！
![](/assets/gradle_task.png)

# 補充

至於為何需要加上Gradle Nature，對Eclipse IDE有什麼影響？
原先引入第三方lib要在Java Build Path一個個加，且相依的jar要自行管控，例如tomcat的jsp-api，比較麻煩。
![](/assets/eclipse_libs.png)

修改build.gradle如下
```groovy
apply plugin: 'java'

repositories {
  mavenCentral() 
}

dependencies {
  implementation 'org.apache.tomcat:tomcat-jsp-api:7.0.109'
}
```

點選「Refresh Gradle Project」後

![](/assets/gradle_sync.png)

再回到「Java Build Path」查看，已將tomcat-jsp-api.jar以及相依的jar都一併引入，超級方便！

![](/assets/eclipse_libs_with_gradle.png)