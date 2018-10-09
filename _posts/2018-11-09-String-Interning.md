---
layout:     post
title:      String Interning - What, Why, and When?
subtitle:   Understanding String Interning
date:       2018-11-09
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
    - Java
---

## 1. What is String Interning?
String Interning is a method of storing only one copy of each distinct String Value, which must be immutable.

In Java, _String_ class has a _public_ method _intern()_ that returns a canonical representation for the string object. Java’s _String_ class privately maintains a pool of strings, where _String_ literals are automatically interned.
> When the intern() method is invoked on a String object it looks the string contained by this String object in the pool, if the string is found there then the string from the pool is returned. Otherwise, this String object is added to the pool and a reference to this String object is returned.

The _intern()_ method helps in comparing two _String_ objects with == operator by looking into the pre-existing pool of string literals, no doubt it is faster than _equals()_ method. The pool of strings in Java is maintained for saving space and for faster comparisons.Normally Java programmers are advised to use _equals()_, not _==_, to compare two strings. This is because _==_ operator compares memory locations, while _equals()_ method compares the content stored in two objects.

## 2. Why and When to Intern ?
Though Java automatically interns all Strings by default, remember that we only need to intern strings when they are not constants, and we want to be able to quickly compare them to other interned strings. The _intern()_ method should be used on strings constructed with _new String()_ in order to compare them by _==_ operator.

Let’s take a look at the following Java program to understand the __*intern()*__ behavior.
```java
public class TestString {
    public static void main(String[] args) {
        String s1 = "Test";
        String s2 = "Test";
        String s3 = new String("Test");
        final String s4 = s3.intern();
        System.out.println(s1 == s2);
        System.out.println(s2 == s3);
        System.out.println(s3 == s4);
        System.out.println(s1 == s3);
        System.out.println(s1 == s4);
        System.out.println(s1.equals(s2));
        System.out.println(s2.equals(s3));
        System.out.println(s3.equals(s4));
        System.out.println(s1.equals(s4));
        System.out.println(s1.equals(s3));
    }
}

//Output
true
false
false
false
true
true
true
true
true
true
```
