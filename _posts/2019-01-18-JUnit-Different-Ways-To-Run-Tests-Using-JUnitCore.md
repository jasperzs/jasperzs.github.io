---
layout:     post
title:      JUnit - Different ways to run tests using JUnitCore
subtitle:   Understanding JUnitCore
date:       2019-01-18
author:     Mr.Humorous 🥘
header-img: img/unitTesting/header.png
catalog: true
tags:
    - Unit Testing
    - JUnit
---

`JUnitCore` class is very useful to run tests from command line or programmatically.

## Using JUnitCore#run(Class<?>... classes) method
`public Result run(Class<?>... classes)`
```java
public class TestExample {

  @Test
  public void test() {
      System.out.println("running test");
  }
}

public class RunClassesExample {

  public static void main(String[] args) {
      JUnitCore jUnitCore = new JUnitCore();
      Result result = jUnitCore.run(TestExample.class);
      Util.printResult(result);
  }
}
```

The returned `Result` object can be used to get failure and success counts.

We are using a Util class to reuse code for others examples.
```java
public class Util {
  public static void printResult(Result result) {
      System.out.printf("Test ran: %s, Failed: %s%n",
              result.getRunCount(), result.getFailureCount());
  }
}
```

Output:
```
running test
Test ran: 1, Failed: 0
```

## Using run(Request request) method
`public Result run(Request request)`
The `Request` class is an abstraction of `Runner` object which performs the task of running tests and fire specific events. Request also supports things like sorting and filtering. We will learn more about Request in other tutorial. Here we are going to see it's very basic use.
```java
public class RunRequestExample {

  public static void main(String[] args) {
      JUnitCore jUnitCore = new JUnitCore();
      Request request = Request.aClass(TestExample.class);
      Result result = jUnitCore.run(request);
      Util.printResult(result);
  }
}
```

Output:
```
running test
Test ran: 1, Failed: 0
```

The static method `Request#aClass(..)` creates a Request that will run all the tests in the provided class.

## Using run(Computer computer, Class<?>... classes) method
`public Result run(Computer computer, Class<?>... classes) `

`Computer` represents a strategy for executing tests. Using it's static method `Computer#serial()` will create an instance of Computer which will run all tests in serial order (one thread). If we use it's subclass: `ParallelComputer`, all tests will be executed in different threads. Note that calling other run(..) methods will execute the tests in serial order.

Following example uses three @Test methods.
```java
public class TestExample2 {

  @Test
  public void test1() {
      printThreadInfo("test1");
  }

  @Test
  public void test2() {
      printThreadInfo("test2");
  }

  @Test
  public void test3() {
      printThreadInfo("test3");
  }

  private static void printThreadInfo(String testName) {
      System.out.printf("Test= %s, Thread= %s, Time= %s%n",
              testName, Thread.currentThread().getName(), LocalTime.now());
      //add some delay
      try {
          TimeUnit.SECONDS.sleep(1);
      } catch (InterruptedException e) {
          e.printStackTrace();
      }
  }
}
```
```java
public class RunComputerExample {
  public static void main(String[] args) {
      runSerial();
      runParallel();
  }

  private static void runSerial() {
      System.out.println("--------\nRunning Serial");
      JUnitCore jUnitCore = new JUnitCore();
      Result run = jUnitCore.run(Computer.serial(), TestExample2.class);
      Util.printResult(run);
  }

  private static void runParallel() {
      System.out.println("--------\nRunning parallel");
      JUnitCore jUnitCore = new JUnitCore();
      Result result = jUnitCore.run(ParallelComputer.methods(),
              TestExample2.class);
      Util.printResult(result);
  }
}
```

Output
```
--------
Running Serial
Test= test1, Thread= main, Time= 19:37:32.616
Test= test2, Thread= main, Time= 19:37:33.670
Test= test3, Thread= main, Time= 19:37:34.670
Test ran: 3, Failed: 0
--------
Running parallel
Test= test2, Thread= pool-1-thread-2, Time= 19:37:35.674
Test= test1, Thread= pool-1-thread-1, Time= 19:37:35.674
Test= test3, Thread= pool-1-thread-3, Time= 19:37:35.674
Test ran: 3, Failed: 0
```

That shows the runParallel() method runs all three test in three different threads in parallel.

## Using static runClasses(Class<?>... classes) method
`public static Result runClasses(Class<?>... classes)`

It's equivalent to using `new JUnitCore().run(Class<?>... classes)`
```java
public class RunClassesStaticExample {
  public static void main(String[] args) {
      Result result = JUnitCore.runClasses(TestExample.class);
      Util.printResult(result);
  }
}
```

Output:
```
running test
Test ran: 1, Failed: 0
```

## Invoking JUnitCore#main method from command line
`java -cp .;<jars-path>  org.junit.runner.JUnitCore [test class name]`

For example, we copied our example class TestExample2.java to a new location (D:\examples\junit-core-test) and then we copied following jars to the root folder:
- C:\Users\Joe\.m2\repository\junit\junit\4.12\junit-4.12.jar
- C:\Users\Joe\.m2\repository\org\hamcrest\hamcrest-core\1.3\hamcrest-core-1.3.jar

We, then, compiled the test class using:
`D:\examples\junit-core-test>javac -d output -cp .;junit-4.12.jar;hamcrest-core-1.3.jar; com/logicbig/example/TestExample2.java`

Now I got following directory structure:
```
D:\examples\junit-core-test>tree /A /F
Folder PATH listing for volume Data
Volume serial number is 00000001 68F9:EDFA
D:.
|   hamcrest-core-1.3.jar
|   junit-4.12.jar
|
+---com
|   \---logicbig
|       \---example
|               TestExample2.java
|
\---output
    \---com
        \---logicbig
            \---example
                    TestExample2.class
```

Finally, I'm going to invoke the tests:
```
D:\examples\junit-core-test>java  -cp .;output;junit-4.12.jar;hamcrest-core-1.3.jar;  org.junit.runner.JUnitCore "com.logicbig.example.TestExample2"
JUnit version 4.12
.Test= test1, Thread= main, Time= 20:52:53.681
.Test= test2, Thread= main, Time= 20:52:54.721
.Test= test3, Thread= main, Time= 20:52:55.721

Time: 3.076

OK (3 tests)
```
