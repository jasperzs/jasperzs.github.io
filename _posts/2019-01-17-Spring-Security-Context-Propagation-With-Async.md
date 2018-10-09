---
layout:     post
title:      ! Spring Security Context Propagation with @Async
subtitle:   Understanding Spring Security Context Propagation
date:       2019-01-17
author:     Mr.Humorous 🥘
header-img: img/springFramework/header.jpg
catalog: true
tags:
    - Spring Framework
    - Spring MVC
---

## 1. Introduction
We are going to focus on the __propagation of the Spring Security principal with *@Async*__. Full example code is on [Github](https://github.com/eugenp/tutorials/tree/master/spring-security-rest).

By default, the Spring Security Authentication is bound to a _ThreadLocal_ – so, when the execution flow runs in a new thread with @Async, that’s not going to be an authenticated context.

That’s not ideal – let’s fix it.

## 2. Maven Dependencies
In order to use the async integration in Spring Security, we need to include the following section in the dependencies of our pom.xml:
```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
    <version>4.2.1.RELEASE</version>
</dependency>
```

## 3. Spring Security Propagation with @Async
```java
@RequestMapping(method = RequestMethod.GET, value = "/async")
@ResponseBody
public Object standardProcessing() throws Exception {
    log.info("Outside the @Async logic - before the async call: "
      + SecurityContextHolder.getContext().getAuthentication().getPrincipal());

    asyncService.asyncCall();

    log.info("Inside the @Async logic - after the async call: "
      + SecurityContextHolder.getContext().getAuthentication().getPrincipal());

    return SecurityContextHolder.getContext().getAuthentication().getPrincipal();
}
```

__We want to check if the Spring SecurityContext is propagated to the new thread__. First, we log the context before the async call, next we run asynchronous method and finally we log the context again. The _asyncCall()_ method has the following implementation:
```java
@Async
@Override
public void asyncCall() {
    log.info("Inside the @Async logic: "
      + SecurityContextHolder.getContext().getAuthentication().getPrincipal());
}
```

## 4. Before the _SecurityContextHolder_ Strategy
Before we’ll set up the _SecurityContextHolder_ strategy, the context inside the _@Async_ method will have a _null_ value.

In particular, if we’ll run the async logic, we’ll be able to log the _Authentication_ object in the main program, but when we’ll log it inside the _@Async_, it’s going to be _null_. This is an example logs output:
```
web - 2016-12-30 22:41:58,916 [http-nio-8081-exec-3] INFO
  o.baeldung.web.service.AsyncService -
  Outside the @Async logic - before the async call:
  org.springframework.security.core.userdetails.User@76507e51:
  Username: temporary; ...

web - 2016-12-30 22:41:58,921 [http-nio-8081-exec-3] INFO
  o.baeldung.web.service.AsyncService -
  Inside the @Async logic - after the async call:
  org.springframework.security.core.userdetails.User@76507e51:
  Username: temporary; ...

web - 2016-12-30 22:41:58,926 [SimpleAsyncTaskExecutor-1] ERROR
o.s.a.i.SimpleAsyncUncaughtExceptionHandler -
Unexpected error occurred invoking async method
'public void org.baeldung.web.service.AsyncServiceImpl.asyncCall()'.
java.lang.NullPointerException: null
```

So, as you can see, inside the executor thread, our call fails with a NPE, as expected – because the Principal isn’t available there.

__To prevent that behaviour, we need to enable the _SecurityContextHolder.MODE_INHERITABLETHREADLOCAL_ strategy:__

`SecurityContextHolder.setStrategyName(SecurityContextHolder.MODE_INHERITABLETHREADLOCAL);`

## 5. After the _SecurityContextHolder_ Strategy
Now, we should have access to the principal inside the async thread, just as we have access to it outside.

Let’s run and have a look at the logging information to make sure that’s the case:
```
web - 2016-12-30 22:45:18,013 [http-nio-8081-exec-3] INFO
  o.baeldung.web.service.AsyncService -
  Outside the @Async logic - before the async call:
  org.springframework.security.core.userdetails.User@76507e51:
  Username: temporary; ...

web - 2016-12-30 22:45:18,018 [http-nio-8081-exec-3] INFO
  o.baeldung.web.service.AsyncService -
  Inside the @Async logic - after the async call:
  org.springframework.security.core.userdetails.User@76507e51:
  Username: temporary; ...

web - 2016-12-30 22:45:18,019 [SimpleAsyncTaskExecutor-1] INFO
  o.baeldung.web.service.AsyncService -
  Inside the @Async logic:
  org.springframework.security.core.userdetails.User@76507e51:
  Username: temporary; ...
```

And here we are – just as we expected, we’re seeing the same principal inside the async executor thread.

## 6. Use Cases
There are a few interesting use cases where we might want to make sure the _SecurityContext_ gets propagated like this:
- we want to make multiple external requests which can run in parallel and which may take significant time to execute
- we have some significant processing to do locally and our external request can execute in parallel to that
- other represent fire-and-forget scenarios, like for example sending an email
