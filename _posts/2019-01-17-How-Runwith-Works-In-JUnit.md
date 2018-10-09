---
layout:     post
title:      JUnit - Understanding how RunWith works
subtitle:   Understanding RunWith
date:       2019-01-17
author:     Mr.Humorous 🥘
header-img: img/unitTesting/header.png
catalog: true
tags:
    - Unit Testing
    - JUnit
---

## 1. Overview
If a JUnit class or its parent class is annotated with [@RunWith](http://junit.org/junit4/javadoc/4.12/org/junit/runner/RunWith.html), JUnit framework invokes the specified class as a test runner instead of running the default runner.

@RunWith has only one element as shown:
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Inherited
public @interface RunWith {
    Class<? extends Runner> value();
}
```

The specified 'value' element must be a subclass of the abstract `org.junit.runner.Runner` class. A Runner class is responsible to run JUnit test, typically by reflection.

An example of @RunWith is [@RunWith(Suite.class)](http://junit.org/junit4/javadoc/4.12/org/junit/runners/Suite.html) which specifies a group of many test classes to run along with the class where it is used.

## 2. Implementing Runner
Here we are implementing abstract methods of Runner class. In run() method, we are invoking the target test methods by reflection.
```java
public class MyRunner extends Runner {

  private Class testClass;
  public MyRunner(Class testClass) {
      this.testClass = testClass;
  }

  @Override
  public Description getDescription() {
      return Description.createTestDescription(testClass, "My runner description");
  }

  @Override
  public void run(RunNotifier notifier) {
      System.out.println("running the tests from MyRunner. " + testClass);
      try {
          Object testObject = testClass.newInstance();
          for (Method method : testClass.getMethods()) {
              if (method.isAnnotationPresent(Test.class)) {
                  notifier.fireTestStarted(Description.EMPTY);
                  method.invoke(testObject);
                  notifier.fireTestFinished(Description.EMPTY);
              }
          }
      } catch (Exception e) {
          throw new RuntimeException(e);
      }

  }
}
```

Note that, we have defined a constructor that takes a Class argument. This is JUnit requirement. During runtime JUnit will pass the target test class to this constructor.

`RunNotifier` is used to fire events to notify the events which the JUnit core layer listens to. These events has information about the test progress.

Let's use the runner in our test class:
```java
@RunWith(MyRunner.class)
public class MyTest {

  @BeforeClass
  public static void beforeClass() {
      System.out.println("in beforeClass method");
  }

  @Before
  public void before() {
      System.out.println("in before method");
  }

  @Test
  public void testMethod1() {
      System.out.println("in the testMethod1");
  }

  @Test
  public void testMethod2() {
      System.out.println("in the testMethod2");
  }
}
```
```
mvn -q test -Dtest=MyTest

Output:
D:\example-projects\junit\junit-run-with>mvn -q test -Dtest=MyTest

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running com.logicbig.example.MyTest
running the tests from MyRunner. class com.logicbig.example.MyTest
in the testMethod2
in the testMethod1
Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.009 sec

Results :

Tests run: 2, Failures: 0, Errors: 0, Skipped: 0
```

In above output, the [lifecycle methods](https://www.logicbig.com/tutorials/unit-testing/junit/lifecycle.html) @BeforeClass and @Before were not invoked. That's our responsibility to invoked these lifecycle methods, just like we invoked @Test methods.

Also it's important to understand that a Runner is invoked by [JUnitCore](https://www.logicbig.com/tutorials/unit-testing/junit/junit-core.html), that means we cannot use JUnitCore.runClasses or similar methods inside Runner#run method, otherwise, that will cause recursive run method calls.

## 3. Other Runner classes
![Runner Classes Hierarchy Illustration]({{ site.url }}/img/unitTesting/JUnit/runWithAnnotation/runnerClassesHierarchy.png)

Instead of extending the low level Runner class, as we did in the last example, we should be extending one of the specialized subclasses of Runner : `ParentRunner` or `BlockJUnit4Runner`.

The abstract ParentRunner runs the tests in a hierarchical manner.

BlockJUnit4Runner is the concrete class. If no @RunWith is specified, then this is the one which runs by default. We will probably be extending this class to customize certain methods. Let's see that with an example.

## 4. Extending BlockJUnit4ClassRunner
```java
public class MyRunner2 extends BlockJUnit4ClassRunner {

  public MyRunner2(Class<?> klass) throws InitializationError {
      super(klass);
  }

  @Override
  protected Statement methodInvoker(FrameworkMethod method, Object test) {
      System.out.println("invoking: " + method.toString());
      return super.methodInvoker(method, test);
  }
}
```
```java
@RunWith(MyRunner2.class)
public class MyTest2 {

  @BeforeClass
  public static void beforeClass() {
      System.out.println("in beforeClass method");
  }

  @Before
  public void before() {
      System.out.println("in before method");
  }

  @Test
  public void testMethod1() {
      System.out.println("in the testMethod1");
  }

  @Test
  public void testMethod2() {
      System.out.println("in the testMethod2");
  }
}
```
```
mvn -q test -Dtest=MyTest2

Output:
D:\example-projects\junit\junit-run-with>mvn -q test -Dtest=MyTest2

-------------------------------------------------------
 T E S T S
-------------------------------------------------------
Running com.logicbig.example.MyTest2
in beforeClass method
invoking: public void com.logicbig.example.MyTest2.testMethod1()
in before method
in the testMethod1
invoking: public void com.logicbig.example.MyTest2.testMethod2()
in before method
in the testMethod2
Tests run: 2, Failures: 0, Errors: 0, Skipped: 0, Time elapsed: 0.032 sec

Results :

Tests run: 2, Failures: 0, Errors: 0, Skipped: 0
```

Above output shows that life-cycle methods, @BeforeClass/@Before, were also called.
