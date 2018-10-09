---
layout:     post
title:      How Lambdas and Anonymous Inner Classes Work
subtitle:   Understanding Lambdas and Anonymous Inner Classes
date:       2018-12-11
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
    - Java
---

## 1. Key Points

- Lambdas implement a functional interface.
- Anonymous Inner Classes can extend a class or implement an interface with any number of methods.
- Variables – Lambdas can only access final or effectively final.
- State – Anonymous inner classes can use instance variables and thus can have state, lambdas cannot.
- Scope – Lambdas can't define a variable with the same name as a variable in enclosing scope.
- Compilation – Anonymous compiles to a class, while lambda is an _invokedynamic_ instruction.

## 2. How They Work

## 2.1 Anonymous Inner Classes (AICs)

- The compiler generates a class file for each anonymous inner class.
    + For example – `AnonymousInnerClass$1.class`
- Like all classes, it needs to be loaded and verified at startup.

## 2.2 Lambdas

The key to lambda implementation is the InvokeDynamic instruction, introduced in Java 7. This allows dynamic languages to bind to symbols at runtime.

A lambda works like this:
- Generates invokedynamic call site and uses a lambdafactory to return the functional implementation.
- Lambda converted to a method to be invoked by invokedynamic.
- The method is stored in a class as a private static method.
- There are two lambda types. Non-capturing lambdas only use fields inside their bodies, whereas capturing lambdas access fields outside their bodies.

### 2.2.1 Non-Capturing Lambda

Doesn't access fields outside its body:
```java
public class NonCapturingLambda {
  public static void main(String[] args) {
    Runnable nonCapturingLambda = () -> System.out.println("NonCapturingLambda");
    nonCapturingLambda.run();
  }
}
```

If we decode the class file using the CFR decompiler, we see the:
- LambdaMetafactory
- Lambda is a static void method in our class

```java
java -jar cfr_0_119.jar NonCapturingLambda --decodelambdas false
/*  * Decompiled with CFR 0_119.  */
import java.io.PrintStream;
import java.lang.invoke.LambdaMetafactory;
public class NonCapturingLambda {
    public static void main(String[] args) {
        Runnable nonCapturingLambda = (Runnable)LambdaMetafactory.metafactory(null,
           null, null, ()V, lambda$0(), ()V)();
        nonCapturingLambda.run();
    }
    private static /* synthetic */ void lambda$0() {
        System.out.println("NonCapturingLambda");
    }
}
```

### 2.2.2 Capturing Lambdas

Accesses final or effectively final fields outside their bodies:
```java
public class CapturingLambda {
    public static void main(String[] args) {
        String effectivelyFinal = "effectivelyFinal";
        Runnable capturingLambda = () -> System.out.println("capturingLambda " + effectivelyFinal);
        capturingLambda.run();
    }
}
```

This decompiles to:
```java

java -jar cfr_0_119.jar CapturingLambda --decodelambdas false
/*  * Decompiled with CFR 0_119.  */
import java.io.PrintStream;
import java.lang.invoke.LambdaMetafactory;
public class CapturingLambda {
    public static void main(String[] args) {
        String effectivelyFinal = "effectivelyFinal";
        Runnable capturingLambda =
          (Runnable)LambdaMetafactory.metafactory(null, null, null, ()V,
                                                  lambda$0(java.lang.String ),
                                                  ()V)((String)effectivelyFinal);
          capturingLambda.run();
    }
    private static /* synthetic */ void lambda$0(String string) {
        System.out.println("capturingLambda " + string);
    }
}
```

The interesting part is that the lambda$0 method signature has gone from empty to taking a parameter String.
