---
layout:     post
title:      The Java Native Keyword and Methods
subtitle:   Introduction to Java Native Keyword
date:       2019-01-21
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
    - Java
---

## 1. The native Keyword in Java
__Simply put, this *native keyword* is a non-access modifier that is used to access methods implemented in a language other than Java like C/C++.__

It indicates platform-dependent implementation of a method or code and also acts as an interface between [JNI](https://www.baeldung.com/jni) and other programming languages.

## 2. native Methods
__A *native* method is a Java method (either an instance method or a class method) whose implementation is also written in another programming language such as C/C++.__

Moreover, a method marked as _native_ cannot have a body and should end with a semicolon:
```java
[ public | protected | private] [return_type] native method ();
```

We can use them to:
- implement an interface with system calls or libraries written in other programming languages
- access system or hardware resources that are only reachable from the other language
- integrate already existing legacy code written in C/C++ into a Java application
- call a compiled dynamically loaded library with arbitrary code from Java

## 3. Examples

### 3.1 Accessing Native Code in Java
A class _DateTimeUtils_ that needs to access a platform-dependent _native_ method named _getSystemTime_:
```java
public class DateTimeUtils {
    public native String getSystemTime();
    // ...
}
```

To load it, we’ll use the _System.loadLibrary_.

Let’s place the call to load this library in a _static_ block so that it is available in our class:
```java
public class DateTimeUtils {
    public native String getSystemTime();

    static {
        System.loadLibrary("nativedatetimeutils");
    }
}
```

We have created a dynamic-link library, _nativedatetimeutils_, that implements _getSystemTime_ in C++ using detailed instructions covered in our [guide to JNI article](https://www.baeldung.com/jni).

### 3.2 Testing native Methods
```java
public class DateTimeUtilsManualTest {

   @BeforeClass
    public static void setUpClass() {
        // .. load other dependent libraries
        System.loadLibrary("nativedatetimeutils");
    }

    @Test
    public void givenNativeLibsLoaded_thenNativeMethodIsAccessible() {
        DateTimeUtils dateTimeUtils = new DateTimeUtils();
        LOG.info("System time is : " + dateTimeUtils.getSystemTime());
        assertNotNull(dateTimeUtils.getSystemTime());
    }
}
```

Below is the output of the logger:
```
[main] INFO  c.b.n.DateTimeUtilsManualTest - System time is : Wed Dec 19 11:34:02 2018
```

As we can see, with the help of the _native_ keyword, we’re successfully able to access a platform-dependent implementation written in another language (in our case C++).
