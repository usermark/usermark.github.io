---
layout: article
title: '[Java] AspectJ入門'
date: 2023-12-28 07:04:37
tags:
- Java
- AOP
- Eclipse
---
實務上遇到的專案使用純JSP開發，是個骨灰級專案，並未導入Sprint。若要使用AOP來做事，就得自行導入。
<!--more-->

# 下載AspectJ library

<https://eclipse.dev/aspectj/downloads.php>
先抓jar檔，這裡視專案使用的Java版本來選定，因專案用到的Java 8，因此選定1.8.14。
![](/assets/download_aspectj_lib.png)

# 安裝Eclipse外掛AJDT

工欲善其事，必先利其器。首先至官網 <https://www.eclipse.org/ajdt/downloads/> 抓取最新版。
![](/assets/ajdt_old.png)

結果看到只有支援到IDE 4.4，且最後發布到2013年就沒有，好了書包收一收可以離開。

開玩笑地，該項目後來遷移到Github上，改來這抓取
<https://github.com/eclipse-aspectj/aspectj/blob/master/docs/developer/IDE.md>
![](/assets/ajdt.png)

打開Eclipse IDE後，Help > Install New Software > Add 輸入如下：
![](/assets/add_ajdt.png)

勾選必要元件後，進行安裝。
![](/assets/add_ajdt_next.png)

待安裝完成後，開始練習寫一個範例程式，順便介紹工具的使用方式。

# 入門

新增專案時，會發現多了AspectJ的資料夾，不用理會，先手動建Java Project，後面會說明如何加上AspectJ模組，好模擬實務上在現有專案導入AOP的情境。
![](/assets/wizard.png)

手動建好DemoA專案後，新增類別
```java
package com.github.usermark;

public class Mike {

    public void doSomething() {
        System.out.println("Mike likes to play sports");
    }
    
    public static void main(String[] args) {
        new Mike().doSomething();
    }
}
```

打開專案的Properties，至Project Natures手動新增AspectJ Nature。
![](/assets/eclipse_add_aspectj.png)

將剛才抓的jar檔用zip編輯器打開，找到aspectjrt.jar後取出。
![](/assets/aj_lib.png)

打開專案的Properties，至Jave Build Path加入aspectjrt.jar。
![](/assets/add_aj_lib.png)

接著新增類別
```java
package com.github.usermark;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;

@Aspect
public class LogAspect {
    
    @Before("execution(* com.github.usermark.*.do*(..))")
    public void addPrint(JoinPoint joinPoint) {
        String className = joinPoint.getTarget().getClass().getName();
        String methodName = joinPoint.getSignature().getName();
        System.out.println(String.format("prepare exec %s.%s", className, methodName));    
    }
}
```

到這裡會好奇，這樣修改有什麼作用。這時AJDT就派上用場，選擇IDE選單 Window > Show View > Other
![](/assets/show_aj_view.png)

找到AspectJ底下的Cross References，選擇Open。
![](/assets/show_aj_view2.png)

同時確認Build Automatically是勾選啟用的。
![](/assets/build_auto.png)

這時就會看到切入點JointPoint是有成功的。
![](/assets/show_aj_view3.png)

實際執行會得到
```
prepare exec com.github.usermark.Mike.doSomething
Mike likes to play sports
```

如果好奇aspect背後到底做了什麼，施了什麼魔法，可打開編譯後的class檔查看，便可得知它只是針對編譯後的class注入程式碼而已。但別小看這樣的行為，它可以減少實際開發要撰寫的程式碼，且更彈性的注入。即使是後續新增的方法，只要有符合其條件，都會做注入。
![](/assets/mike_class_de.png)

# 進階 Inpath

這讓注入第三方jar，即已編譯過的class成為可能。
實際操作一遍，先將DemoA專案打包成jar檔，供其他專案使用。
在專案上按右鍵，選擇Export > Jar file
![](/assets/export_jar.png)

產生jar檔後，接著建立新專案DemoB，引用DemoA.jar。
![](/assets/add_demoa_lib.png)

同樣，打開專案的Properties，至Project Natures手動新增AspectJ Nature。
再來新增類別。
```java
package com.github.usermark;

public class Bill {

    public void doSleep() {
        System.out.println("Bill usually goes to bed at nine");
    }
    
    public static void main(String[] args) {
        new Mike().doSomething();
        new Bill().doSleep();
    }
}
```

先執行看看會得到錯誤
```
Exception in thread "main" java.lang.NoClassDefFoundError: org/aspectj/lang/Signature
    at com.github.usermark.Bill.main(Bill.java:10)
Caused by: java.lang.ClassNotFoundException: org.aspectj.lang.Signature
    at java.net.URLClassLoader.findClass(URLClassLoader.java:387)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:418)
    at sun.misc.Launcher$AppClassLoader.loadClass(Launcher.java:352)
    at java.lang.ClassLoader.loadClass(ClassLoader.java:351)
    ... 1 more
```

因少引入aspectjrt.jar，至Jave Build Path加入aspectjrt.jar後，再次執行
```
prepare exec com.github.usermark.Mike.doSomething
Mike likes to play sports
Bill usually goes to bed at nine
```

接著新增類別
```java
package com.github.usermark;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.After;
import org.aspectj.lang.annotation.Aspect;

@Aspect
public class LogEndAspect {
    
    @After("execution(* com.github.usermark.*.do*(..))")
    public void addPrint(JoinPoint joinPoint) {
        String className = joinPoint.getTarget().getClass().getName();
        String methodName = joinPoint.getSignature().getName();
        System.out.println(String.format("already exec %s.%s", className, methodName));    
    }
}
```

實際執行得到結果
```
prepare exec com.github.usermark.Mike.doSomething
Mike likes to play sports
Bill usually goes to bed at nine
already exec com.github.usermark.Bill.doSleep
```

可以看到第三方的jar裡面的Mike類別並未注入新的LogEndAspect。
打開專案的Properties，至AspectJ Build的Inpath分頁新增DemoA.jar。
![](/assets/inpath.png)

再次執行，可以看到Mike底下多了一行。
```
prepare exec com.github.usermark.Mike.doSomething
Mike likes to play sports
already exec com.github.usermark.Mike.doSomething
Bill usually goes to bed at nine
already exec com.github.usermark.Bill.doSleep
```

# 進階 Aspect Path

這樣Bill類別能否注入jar裡面的LogAspect，當然可以！
打開專案的Properties，至AspectJ Build的Aspect Path分頁新增DemoA.jar。
![](/assets/aspect_path.png)

再次執行，可以看到Bill上面多了一行。
```
prepare exec com.github.usermark.Mike.doSomething
Mike likes to play sports
already exec com.github.usermark.Mike.doSomething
prepare exec com.github.usermark.Bill.doSleep
Bill usually goes to bed at nine
already exec com.github.usermark.Bill.doSleep
```

**參考資料**
1. [AspectJ 使用介绍](https://javadoop.com/post/aspectj)