---
layout:     post
title:      What is Functional Programming?
subtitle:   Understanding Functional Programming
date:       2018-10-23
author:     Mr.Humorous 🥘
header-img: img/functionalProgramming/header.jpg
catalog: true
tags:
    - Functional Programming
    - JavaScript
---

## 1. Overview
__Functional Programming__ is the process of building software by composing __pure functions__, avoiding __shared state__, __mutable data__, and __side-effects__. Functional programming is __declarative__ rather than __imperative__, and application state flows through pure functions.

Functional programming is a __programming paradigm__, meaning that it is a way of thinking about software construction based on some fundamental, defining principles (listed above). Other examples of programming paradigms include object oriented programming and procedural programming.

## 2. Academic Lingos
+ __Pure Function__:
    - A function when given the same inputs, always returns the same output
    - A function has no side-effects
+ __Function Composition__:
    - The process of combining two or more functions in order to produce a new function or perform some computation
        * e.g. `f . g` (the dot means "composed with") = `f(g(x))` in JavaScript
+ __Shared State__:
    - Any variable, object or memory space that exists in a shared scope, or as the property of an object being passed b/w scopes.
    - In OOP, objects are shared b/w scopes by adding properties to other objects.
    - Common Problems:
        * Race condition
        * Changing the order in which functions are called can cause a cascade of failures b/c functions which act on shared state are timing dependent
+ __Immutability__:
    - An __immutable object__ is an object that can’t be modified after it’s created. Conversely, a __mutable object__ is any object which can be modified after it’s created.
+ __Side Effects__:
    - A side effect is any application state change that is observable outside the called function other than its return value. Side effects include:
        * Modifying any external variable or object property (e.g., a global variable, or a variable in the parent function scope chain)
        * Logging to the console
        * Writing to the screen
        * Writing to a file
        * Writing to the network
        * Triggering any external process
        * Calling any other functions with side-effects

## 3. Reusability Through Higher Order Functions
Functional programming tends to reuse a common set of functional utilities to process data. Object oriented programming tends to colocate methods and data in objects. Those colocated methods can only operate on the type of data they were designed to operate on, and often only the data contained in that specific object instance.

In functional programming, any type of data is fair game. The same map() utility can map over objects, strings, numbers, or any other data type because it takes a function as an argument which appropriately handles the given data type. FP pulls off its generic utility trickery using higher order functions.

JavaScript has __first class functions__, which allows us to treat functions as data — assign them to variables, pass them to other functions, return them from functions, etc…

A __higher order__ function is any function which takes a function as an argument, returns a function, or both. Higher order functions are often used to:
+ Abstract or isolate actions, effects, or async flow control using callback functions, promises, monads, etc…
+ Create utilities which can act on a wide variety of data types
+ Partially apply a function to its arguments or create a curried function for the purpose of reuse or function composition
+ Take a list of functions and return some composition of those input functions

## 4. Containers, Functors, Lists, and Streams
A functor is something that can be mapped over. In other words, it’s a container which has an interface which can be used to apply a function to the values inside it. When you see the word functor, you should think “mappable”.

Earlier we learned that the same map() utility can act on a variety of data types. It does that by lifting the mapping operation to work with a functor API. The important flow control operations used by map() take advantage of that interface. In the case of Array.prototype.map(), the container is an array, but other data structures can be functors, too — as long as they supply the mapping API.

Let’s look at how Array.prototype.map() allows you to abstract the data type from the mapping utility to make map() usable with any data type. We’ll create a simple double() mapping that simply multiplies any passed in values by 2:
```javascript
const double = n => n * 2;
const doubleMap = numbers => numbers.map(double);
console.log(doubleMap([2, 3, 4])); // [ 4, 6, 8 ]
```

What if we want to operate on targets in a game to double the number of points they award? All we have to do is make a subtle change to the double() function that we pass into map(), and everything still works:
```javascript
const double = n => n.points * 2;

const doubleMap = numbers => numbers.map(double);

console.log(doubleMap([
  { name: 'ball', points: 2 },
  { name: 'coin', points: 3 },
  { name: 'candy', points: 4}
])); // [ 4, 6, 8 ]
```

Arrays and functors are not the only way this concept of containers and values in containers applies. For example, an array is just a list of things. A list expressed over time is a stream — so you can apply the same kinds of utilities to process streams of incoming events.

## 5. Declarative vs Imperative
Functional programming is a declarative paradigm, meaning that the program logic is expressed without explicitly describing the flow control.

__Imperative__ programs spend lines of code describing the specific steps used to achieve the desired results — the __flow control__: __How__ to do things.
```javascript
const doubleMap = numbers => {
  const doubled = [];
  for (let i = 0; i < numbers.length; i++) {
    doubled.push(numbers[i] * 2);
  }
  return doubled;
};

console.log(doubleMap([2, 3, 4])); // [4, 6, 8]
```

__Imperative__ code frequently utilizes statements. A __statement__ is a piece of code which performs some action. Examples of commonly used statements include for, if, switch, throw, etc…

__Declarative__ programs abstract the flow control process, and instead spend lines of code describing the __data flow__: __What__ to do. The how gets abstracted away.
```javascript
const doubleMap = numbers => numbers.map(n => n * 2);

console.log(doubleMap([2, 3, 4])); // [4, 6, 8]
```

__Declarative__ code relies more on expressions. An __expression__ is a piece of code which evaluates to some value. Expressions are usually some combination of function calls, values, and operators which are evaluated to produce the resulting value.
+ `2 * 2`
+ `doubleMap([2, 3, 4])`
+ `Math.max(4, 3, 2)`

Usually in code, you’ll see expressions being assigned to an identifier, returned from functions, or passed into a function. Before being assigned, returned, or passed, the expression is first evaluated, and the resulting value is used.

## 6. Conclusion
Functional programming favors:
+ Pure functions instead of shared state & side effects
+ Immutability over mutable data
+ Function composition over imperative flow control
+ Lots of generic, reusable utilities that use higher order functions to act on many data types instead of methods that only operate on their colocated data
+ Declarative rather than imperative code (what to do, rather than how to do it)
+ Expressions over statements
+ Containers & higher order functions over ad-hoc polymorphism
    - Ad-hoc polymorphism is a kind of polymorphism in which polymorphic functions can be applied to arguments of different types, because a polymorphic function can denote a number of distinct and potentially heterogeneous implementations depending on the type of argument(s) to which it is applied.
