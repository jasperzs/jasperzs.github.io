---
layout:     post
title:      5 Phases of Java Program
subtitle:   How a Java program is executed
date:       2019-01-21
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
    - Java
---

![5 Phases of Java Program Illustration]({{ site.url }}/img/java/5PhasesOfJavaProgram/5PhasesOfJavaProgram.jpg)

<u>Phase 1: Edit</u>

We create the program on editor, after that it stored in the disk with the name's ending .java

<u>Phase 2: Compile javac (java compiler)</u>

Compiler translate from high-level language program to byte codes and store it in disk with the ending name .class.

<u>Phase 3: Load</u>

Class loader compile read and put those byte codes from disk to Primary Memory.

<u>Phase 4: Verify</u>

Verify byte codes to confirm that all byte codes are valid and do not risk for the Java's security restrictions.

<u>Phase 5: Execute</u>

Java Virtual Machine (JVM) read and translates those byte codes to language that computer can understand (Machine Language). Then execute the program, store it values in primary memory.
