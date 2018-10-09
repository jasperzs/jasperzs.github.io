---
layout:     post
title:      What are CPU and GPU?
subtitle:   Understand what is the difference b/w CPU and GPU and how to avoid bottlenecking
date:       2018-10-29
author:     Mr.Humorous 🥘
header-img: img/functionalProgramming/header.jpg
catalog: true
tags:
    - Functional Programming
    - JavaScript
---

## 1. What is a Function?
A __function__ is a process which takes some input, called __arguments__, and produces some output called a __return value__. Functions may serve the following purposes:
+ __Mapping__: Produce some output based on given inputs. A function __maps__ input values to output values.
```java
Math.max(2, 8, 5); // 8
```
+ __Procedures__: A function may be called to perform a sequence of steps. The sequence is known as a procedure, and programming in this style is known as __procedural programming__.
+ __I/O__: Some functions exist to communicate with other parts of the system, such as the screen, storage, system logs, or network.

## 2. What is a Pure Function?
A __pure function__ is a function which:
+ Given the same input, will always return the same output.
    - Non-pure example:
        * `Math.random(); // => 0.4011148700956255`
        * `Math.random(); // => 0.8533405303023756`
    - Pure example:
        * `highpass(5, 5); // => true`
        * `highpass(5, 5); // => true`
+ Produces no side effects, which means that it can’t alter any external state.
    - Side effects example:
        * Saving value to disk or logging to console
+ Referential Transparency
    - Replace a function call with its resulting value without changing the meaning of the program.
