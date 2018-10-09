---
layout:     post
title:      JUnit - Lifecycle of a Test Class
subtitle:   Understanding JUnit Test Class Lifecycle
date:       2019-01-18
author:     Mr.Humorous 🥘
header-img: img/unitTesting/header.png
catalog: true
tags:
    - Unit Testing
    - JUnit
---

## Annotation Overview
Every time we run a JUnit test class, a new instance of it is created. There is even a new instance before each test method execution. JUnit framework provides following basic lifecycle annotations (all are method level annotations).
- `@BeforeClass`:
  + This annotation must be used on public static void no-arg method.
  + The method is invoked before an instance of this test class is created and any tests are invoked.
  + This life cycle callback is not related to JVM class loading. JUnit framework calls this method explicitly before calling constructor/other methods.
  + This method is called only once, whereas, other instance lifecycle methods are called every time before calling each test method.
  + This annotation is useful for initializing static resources which would, otherwise, be expensive to create during each test invocation.
- `@AfterClass`: Similar to @BeforeClass but is called at the very end of all test/other lifecycle methods. It is called only once. Useful for static resource clean up.
- `@Before`:
  + It should be used on public void no-arg instance method.
  + It is invoked every time before each test method invocation.
  + Used to setup instance variables/resources which can be used during a test method execution. Useful to avoid code duplication and/or when there are many indirection of method calls starting from the target test method.
- `@After`: Similar to @Before but runs after target test method execution. Useful for cleaning up instance resources.
- `@Test`: public void methods to perform tests. This is where we perform one or more assertions by using static methods of `org.junit.Assert`. Assert methods throw `org.junit.AssertionError` on assertion failure. This exception or any other exception is reported as test failure. If no exceptions are thrown then the test will pass.

## Example
```java
public class LifeCycleTest {

  public LifeCycleTest() {
      System.out.printf("Constructor invoked. Instance: %s%n", this);
  }

  @BeforeClass
  public static void beforeClassMethod() {
      System.out.println("@BeforeClass static method invoked.");
  }

  @Test
  public void test1() {
      System.out.printf("@Test method 1  invoked. Instance: %s%n", this);
  }

  @Test
  public void test2() {
      System.out.printf("@Test method 2  invoked. Instance: %s%n", this);
  }

  @Before
  public void beforeMethod() {
      System.out.printf("@Before method invoked. Instance: %s%n", this);
  }

  @After
  public void afterMethod() {
      System.out.printf("@After method invoked. Instance: %s%n", this);
  }

  @AfterClass
  public static void afterClassMethod() {
      System.out.printf("@AfterClass static method invoked.%n");
  }
}
```
```
mvn -q test -Dtest=LifeCycleTest

Output:
D:\example-projects\junit\junit-lifecycle>mvn -q test -Dtest=LifeCycleTest

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running com.logicbig.example.LifeCycleTest
@BeforeClass static method invoked.
Constructor invoked. Instance: com.logicbig.example.LifeCycleTest@26f0a63f
@Before method invoked. Instance: com.logicbig.example.LifeCycleTest@26f0a63f
@Test method 1  invoked. Instance: com.logicbig.example.LifeCycleTest@26f0a63f
@After method invoked. Instance: com.logicbig.example.LifeCycleTest@26f0a63f
Constructor invoked. Instance: com.logicbig.example.LifeCycleTest@71dac704
@Before method invoked. Instance: com.logicbig.example.LifeCycleTest@71dac704
@Test method 2  invoked. Instance: com.logicbig.example.LifeCycleTest@71dac704
@After method invoked. Instance: com.logicbig.example.LifeCycleTest@71dac704
@AfterClass static method invoked.
Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.038 sec

Results :

Tests run: 2, Failures: 0, Errors: 0, Skipped: 0
```

Now let's run the test class LifeCycleTest twice (in the same JVM instance) to confirm `@BeforeClass` and `@AfterClass` run each time instead of once (to see if that is not related to JVM class loading). We can run tests twice programmatically (in the same JVM instance) by using `JUnitCore`:
```java
public class ProgrammaticTestRunner {

  public static void main(String[] args) {
      JUnitCore junit = new JUnitCore();
      junit.run(LifeCycleTest.class);
      System.out.println("----------------");
      junit.run(LifeCycleTest.class);
  }
}
```
```
Output:
@BeforeClass static method invoked.
Constructor invoked. Instance: com.logicbig.example.LifeCycleTest@4a83c530
@Before method invoked. Instance: com.logicbig.example.LifeCycleTest@4a83c530
@Test method 1  invoked. Instance: com.logicbig.example.LifeCycleTest@4a83c530
@After method invoked. Instance: com.logicbig.example.LifeCycleTest@4a83c530
Constructor invoked. Instance: com.logicbig.example.LifeCycleTest@63446401
@Before method invoked. Instance: com.logicbig.example.LifeCycleTest@63446401
@Test method 2  invoked. Instance: com.logicbig.example.LifeCycleTest@63446401
@After method invoked. Instance: com.logicbig.example.LifeCycleTest@63446401
@AfterClass static method invoked.
----------------
@BeforeClass static method invoked.
Constructor invoked. Instance: com.logicbig.example.LifeCycleTest@49a46be8
@Before method invoked. Instance: com.logicbig.example.LifeCycleTest@49a46be8
@Test method 1  invoked. Instance: com.logicbig.example.LifeCycleTest@49a46be8
@After method invoked. Instance: com.logicbig.example.LifeCycleTest@49a46be8
Constructor invoked. Instance: com.logicbig.example.LifeCycleTest@9806ab1
@Before method invoked. Instance: com.logicbig.example.LifeCycleTest@9806ab1
@Test method 2  invoked. Instance: com.logicbig.example.LifeCycleTest@9806ab1
@After method invoked. Instance: com.logicbig.example.LifeCycleTest@9806ab1
@AfterClass static method invoked.
```
