---
layout:     post
title:      Generic Array Creation
subtitle:   How to get around of generic array creation?
date:       2018-11-19
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
    - Java
    - Data Structure
---

## 1. Problem
Java does not allow you to create arrays of generic classes:
```java
Set<String>[] sets = new Set<String>[5];
```

## 2. Workaround 1: Raw types
```java
@SuppressWarnings("unchecked")
Set<String>[] sets = new Set[5];
```

## 3. Workaround 2: Use a List instead
```java
List<Set<String>> sets = new ArrayList<>();
```

## 4. Workaround 3: Non-generic subclass
You can create a non-generic subclass (local if you like) that extends the generic class:
```java
class StringHashSet extends HashSet<String> {}
Set<String>[] sets = new StringHashSet[5];
```

## 5. What’s the reason for this limitation?
Let’s illustrate what could happen with an example:
```java
// Suppose this was allowed
Set<String>[] arr = new Set<String>[5];

// Arrays are covariant:
Object[] oa = arr;

// Putting a Set<Integer> in arr!
oa[0] = Collections.singleton(3);

// Looks type safe but isn't!
String s = arr[0].iterator().next();
```
Arrays actually have the same problem. This program __does__ in fact even compile:
```java
String[] arr = new String[5];
Object[] oa = arr;
oa[0] = 5; // Eeek! An Integer in a String[]?
String s = arr[0];
```
Here however, `oa[0] = 5` throws an `ArrayStoreException` in runtime. In other words, when generics were introduced the designers traded some expressiveness for some added compile time type safety.
