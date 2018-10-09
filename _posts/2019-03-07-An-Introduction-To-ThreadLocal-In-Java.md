---
layout:     post
title:      An Introduction to ThreadLocal in Java
subtitle:   Understanding ThreadLocal
date:       2019-03-07
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
    - Java
---

## 1. Overview
ThreadLocal allows storing data individually for the current thread and simply wrap that data within a special type of object.

## 2. ThreadLocal API
The _TheadLocal_ construct allows us to store data that will be __accessible only__ by __a specific thread__.

Let’s say that we want to have an _Integer_ value that will be bundled with the specific thread:
```java
ThreadLocal<Integer> threadLocalValue = new ThreadLocal<>();
```

Next, when we want to use this value from a thread we only need to call a _get()_ or _set()_ method. Simply put, we can think that _ThreadLocal_ stores data inside of a map – with the thread as the key.

Due to that fact, when we call a _get()_ method on the _threadLocalValue_ we will get an _Integer_ value for the requesting thread:
```java
threadLocalValue.set(1);
Integer result = threadLocalValue.get();
```

We can construct an instance of the _ThreadLocal_ by using the _withInitial()_ static method and passing a supplier to it:
```java
ThreadLocal<Integer> threadLocal = ThreadLocal.withInitial(() -> 1);
```

To remove value from the _ThreadLocal_ we can call a _remove()_ method:
```java
threadLocal.remove()
```

To see how to use the _ThreadLocal_ properly, firstly, we will look at an example that does not use a _ThreadLocal_, then we will rewrite our example to leverage that construct.

## 3. Storing User Data in a Map
Let’s consider a program that needs to store the user specific _Context_ data per given user id:
```java
public class Context {
    private String userName;

    public Context(String userName) {
        this.userName = userName;
    }
}
```

We want to have one thread per user id. We’ll create a _SharedMapWithUserContext_ class that implements a _Runnable_ interface. The implementation in the _run()_ method calls some database through the _UserRepository_ class that returns a _Context_ object for a given _userId_.

Next, we store that context in the _ConcurentHashMap_ keyed by _userId_:
```java
public class SharedMapWithUserContext implements Runnable {

    public static Map<Integer, Context> userContextPerUserId
      = new ConcurrentHashMap<>();
    private Integer userId;
    private UserRepository userRepository = new UserRepository();

    @Override
    public void run() {
        String userName = userRepository.getUserNameForUserId(userId);
        userContextPerUserId.put(userId, new Context(userName));
    }

    // standard constructor
}
```

We can easily test our code by creating and starting two threads for two different _userIds_ and asserting that we have two entries in the _userContextPerUserId_ map:
```java
SharedMapWithUserContext firstUser = new SharedMapWithUserContext(1);
SharedMapWithUserContext secondUser = new SharedMapWithUserContext(2);
new Thread(firstUser).start();
new Thread(secondUser).start();

assertEquals(SharedMapWithUserContext.userContextPerUserId.size(), 2);
```

## 4. Storing User Data in ThreadLocal
We can rewrite our example to store the user _Context_ instance using a _ThreadLocal_. Each thread will have its own _ThreadLocal_ instance.

When using _ThreadLocal_ we need to be very careful because every _ThreadLocal_ instance is associated with a particular thread. In our example, we have a dedicated thread for each particular _userId_ and this thread is created by us so we have full control over it.

The _run()_ method will fetch the user context and store it into the _ThreadLocal_ variable using the _set()_ method:
```java
public class ThreadLocalWithUserContext implements Runnable {

    private static ThreadLocal<Context> userContext
      = new ThreadLocal<>();
    private Integer userId;
    private UserRepository userRepository = new UserRepository();

    @Override
    public void run() {
        String userName = userRepository.getUserNameForUserId(userId);
        userContext.set(new Context(userName));
        System.out.println("thread context for given userId: "
          + userId + " is: " + userContext.get());
    }

    // standard constructor
}
```

We can test it by starting two threads that will execute the action for a given _userId_:
```java
ThreadLocalWithUserContext firstUser
  = new ThreadLocalWithUserContext(1);
ThreadLocalWithUserContext secondUser
  = new ThreadLocalWithUserContext(2);
new Thread(firstUser).start();
new Thread(secondUser).start();
```

After running this code we’ll see on the standard output that _ThreadLocal_ was set per given thread:
```
thread context for given userId: 1 is: Context{userNameSecret='18a78f8e-24d2-4abf-91d6-79eaa198123f'}
thread context for given userId: 2 is: Context{userNameSecret='e19f6a0a-253e-423e-8b2b-bca1f471ae5c'}
```

We can see that each of the users has its own Context.

## 5. Do not use ThreadLocal with ExecutorService
If we want to use an _ExecutorService_ and submit a _Runnable_ to it, using _ThreadLocal_ will yield non-deterministic results – because we do not have a guarantee that every _Runnable_ action for a given _userId_ will be handled by the same thread every time it is executed.

Because of that, our _ThreadLocal_ will be shared among different _userIds_. That’s why we should not use a _TheadLocal_ together with _ExecutorService_. It should only be used when we have full control over which thread will pick which runnable action to execute.

## 6. Conclusion
In this quick article, we were looking at the _ThreadLocal_ construct. We implemented the logic that uses _ConcurrentHashMap_ that was shared between threads to store the context associated with a particular _userId_. Next, we rewrote our example to leverage _ThreadLocal_ to store data that is associated with a particular _userId_ and with a particular thread.

The implementation of all these examples and code snippets can be found in the [GitHub project](https://github.com/eugenp/tutorials/tree/master/core-java-concurrency-advanced) – this is a Maven project, so it should be easy to import and run as it is.
