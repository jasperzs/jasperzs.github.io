---
layout:     post
title:      What is JVM, JRE and JDK?
subtitle:   Understanding the relationship between JVM, JRE and JDK
date:       2018-10-13
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
    - Java
---

## 1. Relationship Overview
![JVM JRE JDK Relationship Overview]({{ site.url }}/img/java/jvmJreJdk/relationshipOverview.jpg)

### 1.1 What is Java Web Start?
Java Web Start software provides the power to launch full-featured applications with a single click. Users can download and launch applications, such as a complete spreadsheet program or an Internet chat client, without going through lengthy installation procedures.

With Java Web Start software, users can launch a Java application by clicking a link in a web page. The link points to a Java Network Launch Protocol (JNLP) file, which instructs Java Web Start software to download, cache, and run the application.

Java Web Start software provides Java developers and users with many deployment advantages:

+ With Java Web Start software, you can place a single Java application on a web server for deployment to a wide variety of platforms, including Windows, Linux, and Solaris.
+ Java Web Start software supports multiple, simultaneous versions of the Java platform.
  An application can request a specific version of the Java Runtime Environment (JRE) software without conflicting with the needs of other applications.
+ Users can create a desktop shortcut to launch a Java Web Start application outside a browser.
+ Java Web Start software takes advantage of the inherent security of the Java platform. By default, applications have restricted access to local disk and network resources.
+ Applications launched with Java Web Start software are cached locally for improved performance.
+ Updates to a Java Web Start application are automatically downloaded when the application is run standalone from the user's desktop.

Java Web Start software is installed as part of the JRE software. Users do not have to install Java Web Start software separately or perform additional tasks to use Java Web Start applications.

## 2. What is JVM?
在上圖中，Platforms表示Solaris、Linux、Windows各種作業系統平台，在這些平台上架構了Java Virtual Machne，也就是JVM，JVM讓Java可以跨平台，但是跨平台是怎麼一回事？在這之前，你得先了解不能跨平台是怎麼一回事。

對於電腦而言，只認識一種語言，也就是0、1序列組成的機器指令。當你使用C/C++等高階語言撰寫程式時，其實這些語言，是比較貼近人類可閱讀的文法，也就是比較接近英語文法的語言。這是為了方便人類閱讀及撰寫，電腦其實看不懂C/C++這類語言，為了將C/C++翻譯為0、1序列組成的機器指令，你必須有個翻譯員，擔任翻譯員工作的就是編譯器(Compiler)。

問題在於，每個平台認識的0、1序列並不一樣。某個指令在Windows上也許是0101，在Linux下也許是1010，因此必須使用不同的編譯器為不同平台編譯出可執行的機器碼，在Windows平台上編譯好的程式，不能直接拿到Linux等其它平台執行，也就是說，你的應用程式無法達到「編譯一次，到處執行」的跨平台目的。

Java 是個高階語言，要讓電腦執行你撰寫的程式，也得透過編譯器的翻譯。不過Java編譯時，並不直接編譯為相依於某平台的0、1序列，而是翻譯為中介格式的位元碼(Byte code)。

Java 原始碼副檔名為*.java，經過編譯器翻譯為副檔名*.class的位元碼。如果想要執行位元碼檔案，目標平台必須安裝JVM(Java Virtual Machine)。JVM會將位元碼翻譯為相依於平台的機器碼。

不同的平台必須安裝專屬該平台的JVM。這就好比你講中文(*.java)，Java編譯器幫你翻譯為英語(*.class)，之後這份英語文件，到各國家之後，再由當地看得懂英文的人(JVM)翻譯為當地語言(機器碼)。

所以JVM擔任的職責之一就是當地翻譯員，將位元碼檔案翻譯為當時平台看得懂的0、1序列，有了JVM，你的Java程式就可以達到「編譯一次，到處到處執行」的跨平台目的。除了瞭解JVM具有讓Java程式跨平台的重要任務之外，撰寫Java程式時，對JVM的重要認知就是：

__對Java程式而言，只認識一種作業系統，這個系統叫JVM，位元碼檔案(副檔名為.class的檔案)就是JVM的可執行檔。__

Java程式理想上，並不用理會真正執行於哪個平台，只要知道如何執行於JVM就可以了，至於JVM實際上如何與底層平台作溝通，則是JVM自己的事！由於JVM實際上就相當於Java程式的作業系統，JVM就負責了Java程式的各種資源管理。

## 2. What is JRE?
Java是個標準，`System.out.println("Hello World");`, System、out、println這些名稱，都是標準中所規範的名稱，實際上必須要有人根據標準撰寫出System.java，編譯為System.class，如此你才能在撰寫第一個Java程式時，使用System類別(Class)上out物件(Object)的println()方法(Method)。

誰來實作System.java？誰來編譯為.class？可能是Oracle、IBM、Apache，無論如何，這些廠商必須根據相關的JSR標準文件，將標準程式庫實作出來，如此你撰寫的第一個Java程式，在Oracle、IBM、Apache等廠商實作的JVM上運行時，引用如System這些標準API，你的第一個Java程式，才可能輕易地運行在不同的平台。

在上圖中右邊可以看到Java SE API，涵蓋了各式常用的程式庫，像是通用的群集(Collection)、輸入輸出、連線資料庫的JDBC、撰寫視窗程式的AWT與Swing等，這些都是在各個JSR標準文件規範之中，

Java Runtime Environment就是Java執行環境，簡稱JRE，包括了Java SE API與JVM。只要你使用Java SE API中的程式庫，在裝有JRE的電腦上就可以直接運行，無需額外在你的程式中再包裝標準程式庫，而可以由JRE直接提供。

## 3. What is JDK?
先前說過，你要在.java中撰寫Java程式語言，使用編譯器編譯為.class檔案，那麼像編譯器這樣的工具程式是由誰提供？答案就是JDK，全名為Java Development Kit！

正如上圖所示，JDK包括了javac、appletviewer、javadoc等工具程式，對於要開發Java程式的人，必須安裝的是JDK，如此才有這些工具程式可以使用，JDK本身包括了JRE，如此你才能執行Java程式，所以總結就是「JDK包括了Java程式語言、工具程式與JRE，JRE則包括了部署技術、Java SE API與JVM」。

撰寫Java程式的人才需要JDK，如果你的程式只是想讓朋友執行呢？那他只要裝JRE就可以了，不用安裝JDK，因為他不需要javac這些工具程式，但他需要Java SE API與JVM。
