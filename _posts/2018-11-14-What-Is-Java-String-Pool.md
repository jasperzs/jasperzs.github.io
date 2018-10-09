---
layout:     post
title:      What is Java String Pool?
subtitle:   Understanding String Pool
date:       2018-11-14
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
    - Java
---

As the name suggests, __String Pool__ in java is a pool of Strings stored in __Java Heap Memory__. We know that String is special class in java and we can create String object using new operator as well as providing values in double quotes.

## String Pool in Java
Here is a diagram which clearly explains how String Pool is maintained in java heap space and what happens when we use different ways to create Strings.
![How String Pool Looks Like In Java Heap]({{ site.url }}/img/java/stringPool/howStringPoolLooksLikeInJavaHeap.png)

String Pool is possible only because String is immutable in Java and it’s implementation of String interning concept. String pool is also example of Flyweight design pattern.

String pool helps in saving a lot of space for Java Runtime although it takes more time to create the String.

When we use double quotes to create a String, it first looks for String with same value in the String pool, if found it just returns the reference else it creates a new String in the pool and then returns the reference.

However using new operator, we force String class to create a new String object in heap space. We can use __intern()__ method to put it into the pool or refer to other String object from string pool having same value.

Here is the java program for the String Pool image:
```java
package com.journaldev.util;

public class StringPool {

    /**
     * Java String Pool example
     * @param args
     */
    public static void main(String[] args) {
        String s1 = "Cat";
        String s2 = "Cat";
        String s3 = new String("Cat");

        System.out.println("s1 == s2 :"+(s1==s2));
        System.out.println("s1 == s3 :"+(s1==s3));
    }

}
```

Output of the above program is:
```java
s1 == s2 :true
s1 == s3 :false
```

How many string is getting created in below statement:
```java
String str = new String("Cat");
```

In above statement, either 1 or 2 string will be created. If there is already a string literal “Cat” in the pool, then only one string “str” will be created in the heap space. If there is no string literal “Cat” in the pool, then it will be first created in the pool and then in the heap space, so total 2 string objects will be created.
