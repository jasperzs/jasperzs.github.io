---
layout:     post
title:      Type Erasure in Java Explained
subtitle:   Examples of Type Erasure
date:       2019-03-14
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
    - Java
---

## 1. Overview
In this quick article, we’ll discuss the basics of an important mechanism in Java’s generics known as type erasure.

## 2. What is Type Erasure?
__Type erasure can be explained as the process of enforcing type constraints only at compile time and discarding the element type information at runtime.__

For example:
```java
public static  <E> boolean containsElement(E [] elements, E element){
    for (E e : elements){
        if(e.equals(element)){
            return true;
        }
    }
    return false;
}
```

When compiled, the unbound type _E_ gets replaced with an actual type of _Object_:
```java
public static  boolean containsElement(Object [] elements, Object element){
    for (Object e : elements){
        if(e.equals(element)){
            return true;
        }
    }
    return false;
}
```

The compiler ensures type safety of our code and prevents runtime errors.

## 3. Types of Type Erasure
Type erasure can occur at class (or variable) and method levels.

### 3.1. Class Type Erasure
At the class level, type parameters on the class are discarded during code compilation and replaced with its first bound, or _Object_ if the type parameter is unbound.

Let’s implement a _Stack_ using an array:
```java
public class Stack<E> {
    private E[] stackContent;

    public Stack(int capacity) {
        this.stackContent = (E[]) new Object[capacity];
    }

    public void push(E data) {
        // ..
    }

    public E pop() {
        // ..
    }
}
```

Upon compilation, the unbound type parameter _E_ is replaced with _Object_:
```java
public class Stack {
    private Object[] stackContent;

    public Stack(int capacity) {
        this.stackContent = (Object[]) new Object[capacity];
    }

    public void push(Object data) {
        // ..
    }

    public Object pop() {
        // ..
    }
}
```

In a case where the type parameter _E_ is bound:
```java
public class BoundStack<E extends Comparable<E>> {
    private E[] stackContent;

    public BoundStack(int capacity) {
        this.stackContent = (E[]) new Object[capacity];
    }

    public void push(E data) {
        // ..
    }

    public E pop() {
        // ..
    }
}
```

When compiled, the bound type parameter _E_ is replaced with the first bound class, _Comparable_ in this case:
```java
public class BoundStack {
    private Comparable [] stackContent;

    public BoundStack(int capacity) {
        this.stackContent = (Comparable[]) new Object[capacity];
    }

    public void push(Comparable data) {
        // ..
    }

    public Comparable pop() {
        // ..
    }
}
```

### 3.2. Method Type Erasure
For method-level type erasure, the method’s type parameter is not stored but rather converted to its parent type _Object_ if it’s unbound or it’s first bound class when it’s bound.

Let’s consider a method to display the contents of any given array:
```java
public static <E> void printArray(E[] array) {
    for (E element : array) {
        System.out.printf("%s ", element);
    }
}
```

Upon compilation, the type parameter E is replaced with Object:
```java
public static void printArray(Object[] array) {
    for (Object element : array) {
        System.out.printf("%s ", element);
    }
}
```

For a bound method type parameter:
```java
public static <E extends Comparable<E>> void printArray(E[] array) {
    for (E element : array) {
        System.out.printf("%s ", element);
    }
}
```

We’ll have the type parameter _E_ erased and replaced with _Comparable_:
```java
public static void printArray(Comparable[] array) {
    for (Comparable element : array) {
        System.out.printf("%s ", element);
    }
}
```

## 4. Edge Cases
Sometime during the type erasure process, the compiler creates a synthetic method to differentiate similar methods. These may come from method signatures extending the same first bound class.

Let’s create a new class that extends our previous implementation of _Stack_:
```java
public class IntegerStack extends Stack<Integer> {

    public IntegerStack(int capacity) {
        super(capacity);
    }

    public void push(Integer value) {
        super.push(value);
    }
}
```

Now let’s look at the following code:
```java
IntegerStack integerStack = new IntegerStack(5);
Stack stack = integerStack;
stack.push("Hello");
Integer data = integerStack.pop();
```

After type erasure, we have:
```java
IntegerStack integerStack = new IntegerStack(5);
Stack stack = (IntegerStack) integerStack;
stack.push("Hello");
Integer data = (String) integerStack.pop();
```

Notice how we can push a _String_ on the _IntegerStack_ – because _IntegerStack_ inherited _push(Object)_ from the parent class _Stack_. This is, of course, incorrect – as it should be an integer since _integerStack_ is a _Stack&lt;Integer&gt;_ type.

So, not surprisingly, an attempt to _pop_ a _String_ and assign to an _Integer_ causes a _ClassCastException_ from a cast inserted during the _push_ by the compiler.

### 4.1. Bridge Methods
To solve the edge case above, the compiler sometimes creates a bridge method. This is a synthetic method created by the Java compiler while compiling a class or interface that extends a parameterized class or implements a parameterized interface where method signatures may be slightly different or ambiguous.

In our example above, the Java compiler preserves polymorphism of generic types after erasure by ensuring no method signature mismatch between _IntegerStack‘s push(Integer)_ method and _Stack‘s push(Object)_ method.

Hence, the compiler creates a bridge method here:
```java
public class IntegerStack extends Stack {
    // Bridge method generated by the compiler

    public void push(Object value) {
        push((Integer)value);
    }

    public void push(Integer value) {
        super.push(value);
    }
}
```

Consequently, _Stack_ class’s _push_ method after type erasure, delegates to the original _push_ method of _IntegerStack_ class.

## 5. Conclusion
In this tutorial, we’ve discussed the concept of type erasure with examples in type parameter variables and methods.

You can read more about these concepts:
- [Java Language Specification: Type Erasure](https://docs.oracle.com/javase/specs/jls/se8/html/jls-4.html#jls-4.6)
- [The Basics of Java Generics](https://www.baeldung.com/java-generics)

As always, the source code that accompanies this article is available over on [GitHub](https://github.com/eugenp/tutorials/tree/master/core-java-lang-oop).
