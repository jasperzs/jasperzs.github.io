---
layout:     post
title:      What is a Java Statement?
subtitle:   Understanding Java Statement
date:       2018-10-12
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
    - Java
---

## 1. Defition of Java Statement
Statements are similar to sentences in the English language. A sentence forms a complete idea which can include one or more clauses. Likewise, a statement in Java forms a complete command to be executed and can include one or more expressions.

<ins>*In simpler terms, a Java statement is just an instruction that explains what should happen.*</ins>

## 2. Types of Java Statement
+ __Expression statements__ change values of variables, call methods, and create objects.
+ __Declaration statements__ declare variables.
+ __Control-flow statements__ determine the order that statements are executed. Typically, Java statements parse from the top to the bottom of the program. However, with control-flow statements, that order can be interrupted to implement branching or looping so that the Java program can run particular sections of code based on certain conditions.
```java
//declaration statement
int number;
//expression statement
number = 4;
//control flow statement
if (number < 10 )
{
  //expression statement
  System.out.println(number + " is less than ten");
}
```
