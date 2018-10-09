---
layout:     post
title:      Arrays vs. ArrayLists
subtitle:   Understanding when to choose Arrays vs. ArrayLists
date:       2018-11-19
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
    - Java
    - Data Structure
---

## 1. Arrays vs ArrayLists (and other Lists)
An array (something like `int[]`) is a built in type while `ArrayList` is a regular class part of the Java standard library.

## 2. When to use which?
Sometimes you __must__ use an array. For example:
- An API method takes an array as argument or returns an array
- You need to work with primitives for performance reasons
Unless you have a specific reason to use an array (such as those mentioned above), use a `List`, such as an `ArrayList`.

## 3. Resizing
Once you’ve created an array, it __can’t be resized__. You can for instance not append an element to the end of an array, or remove an element.

Most list types (including `ArrayList`) provide `List.add` and `List.remove` which allows it to __grow and shrink__.

## 4. Insertion
Both arrays and lists allow you to update the content at a specific index.
```java
arr[5] = 17;
list.set(5, 17);
```
Lists however, also allows you to __insert__ an element in the middle, shifting all subsequent elements from index i to i+1.
```java
list.add(5, 17);  // insert 17 at index 5
```

## 5. Primitives
An arrray can store primitive values. Lists can not. You can have a `List` of integers, but you’ll have to use `List<Integer>`. `List<int>` doesn’t work.

Autoboxing: Luckily there's something called autoboxing which silently transforms an `int` to an `Integer` behind the scenes. So despite the fact that a `List` can only hold `Integer` values, we can still write `list.add(5)`;.

## 6. Generic elements
You can’t create an array of generic elements. This won’t compile:
```java
// Error: generic array creation
Set<String>[] sets = new Set<String>[3];
```
This however works fine:
```java
List<Set<String>> sets = new ArrayList<Set<String>>();
```
There is however a simple workaround for the array case. See Java: [Generic Array Creation]({{ site.url }}/2018/11/19/Generic-Array-Creation/).

## 7. Immutability
There’s no way to make the elements of an array “final”. If you write for instance
```java
final int[] arr = { 1, 2, 3 };
```
Something like `arr[1] = 77` is still allowed.
Element immutability can be achieved for lists by doing:
```java
List<Integer> list = Collections.unmodifiableList(Arrays.asList(1, 2, 3));
```

## 8. Covariance
A `String[]` is a subtype of `Object[]`. This allows us to write:
```java
String[] strings = new String[1];
Object[] objects = strings;
objects[0] = new Object();
```
Although it throws an exception when executed!
A `List<String>` is however not a subtype of a `List<Object>`. This for example does not compile:
```java
List<String> strings = new ArrayList<>();
List<Object> objects = strings;  // Error: Incompatible types
objects.add(new Object());
```
Put in fancy computer science terms: Arrays are covariant, while lists are not.

## 9. List methods
You can’t use `List` methods on arrays… __WRONG__. There’s in fact a bridge from the array world over to the `List` world. The bridge is called `Arrays.asList`. This method returns a `List` implementation backed by the provided array, which means that modifications to the list affect the original array.

Here’s an example using `List.replaceAll` on an array:
```java
String[] strs = { "a", "b", "c", "d" };
Arrays.asList(strs).replaceAll(s -> s + s);
// strs == { "aa", "bb", "cc", "dd" }
```
