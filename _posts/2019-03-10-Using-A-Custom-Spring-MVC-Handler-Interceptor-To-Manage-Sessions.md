---
layout:     post
title:      Using a Custom Spring MVC’s Handler Interceptor to Manage Sessions
subtitle:   Customize Handler Interceptor to Manage Sessions
date:       2019-03-10
author:     Mr.Humorous 🥘
header-img: img/springFramework/header.jpg
catalog: true
tags:
    - Spring Framework
    - Spring MVC
---

## 1. Introduction
In this tutorial, we’ll use Spring MVC's handler interceptor to emulate a __session timeout logic__ by setting custom counters and tracking sessions manually.

To understand the basics of handler interceptor, please refer to this [article](https://www.baeldung.com/spring-mvc-handlerinterceptor).

## 2. Maven Dependencies
```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>5.0.6.RELEASE</version>
</dependency>
```

This dependency only covers Spring Web so don’t forget to add spring-core and spring-context for a full (minimal) web application.

## 3. Custom Implementation of Session Timeouts
In this example, we will configure maximum inactive time for the users in our system. After that time, they will be logged out automatically from the application.

This logic is just a __proof of concept__ – we can of course easily achieve the same result using session timeouts – but the result is not the point here, the usage of the interceptor is.

And so, we want to make sure that session will be invalidated if the user is not active. For example, if a user forgot to log out, the inactive time counter will prevent accessing the account by unauthorized users. In order to do that, we need to set constant for the inactive time:
```java
private static final long MAX_INACTIVE_SESSION_TIME = 5 * 10000;
```

We set it to 50 seconds for testing purposes; don’t forget, it is counted in ms.

Now, we need to keep track of each session in our app, so we need to include this Spring Interface:
```java
@Autowired
private HttpSession session;
```

Let’s proceed with the _preHandle()_ method.

### 3.1. preHandle()
In this method we will include following operations:
- setting timers to check handling time of the requests
- checking if a user is logged in (using _UserInterceptor_ method from this [article](https://www.baeldung.com/spring-model-parameters-with-handler-interceptor))
- automatic logging out, if the user’s inactive session time exceeds maximum allowed value

Let’s look at the implementation:
```java
@Override
public boolean preHandle(
  HttpServletRequest req, HttpServletResponse res, Object handler) throws Exception {
    log.info("Pre handle method - check handling start time");
    long startTime = System.currentTimeMillis();
    request.setAttribute("executionTime", startTime);
}
```

In this part of the code, we set the _startTime_ of handling execution. From this moment, we will count a number of seconds to finish handling of each request. In the next part, we will provide logic for session time, only if somebody logged in during his HTTP Session:
```java
if (UserInterceptor.isUserLogged()) {
    session = request.getSession();
    log.info("Time since last request in this session: {} ms",
      System.currentTimeMillis() - request.getSession().getLastAccessedTime());
    if (System.currentTimeMillis() - session.getLastAccessedTime()
      > MAX_INACTIVE_SESSION_TIME) {
        log.warn("Logging out, due to inactive session");
        SecurityContextHolder.clearContext();
        request.logout();
        response.sendRedirect("/spring-rest-full/logout");
    }
}
return true;
```

First, we need to get the session from the request.

Next, we do some console logging, about who is logged in, and how long has passed, since the user performs any operation in our application. We may use _session.getLastAccessedTime()_ to obtain this information, subtract it from current time and compare with our _MAX_INACTIVE_SESSION_TIME_.

If time is longer than we allow, we clear the context, log out the request and then (optionally) send a redirect as a response to default logout view, which is declared in Spring Security configuration file.

To complete counters for handling time example, we also implement _postHandle()_ method, which is described in the next subsection.

### 3.2. postHandle()
This method is implementation just to get information, how long it took to process the current request. As you saw in the previous code snippet, we set _executionTime_ in Spring model. Now it’s time to use it:
```java
@Override
public void postHandle(
  HttpServletRequest request, 
  HttpServletResponse response,
  Object handler, 
  ModelAndView model) throws Exception {
    log.info("Post handle method - check execution time of handling");
    long startTime = (Long) request.getAttribute("executionTime");
    log.info("Execution time for handling the request was: {} ms",
      System.currentTimeMillis() - startTime);
}
```

The implementation is simple – we check an execution time and subtract it from a current system time. Just remember to cast the value of the model to _long_.

Now we can log execution time properly.

## 4. Config of the Interceptor
To add our newly created _Interceptor_ into Spring configuration, we need to override _addInterceptors()_ method inside _WebConfig_ class that implements _WebMvcConfigurer_:
```java
@Override
public void addInterceptors(InterceptorRegistry registry) {
    registry.addInterceptor(new SessionTimerInterceptor());
}
```

We may achieve the same configuration by editing our XML Spring configuration file:
```xml
<mvc:interceptors>
    <bean id="sessionTimerInterceptor" class="org.baeldung.web.interceptor.SessionTimerInterceptor"/>
</mvc:interceptors>
```

Moreover, we need to add listener, in order to automate the creation of the _ApplicationContext_:
```java
public class ListenerConfig implements WebApplicationInitializer {
    @Override
    public void onStartup(ServletContext sc) throws ServletException {
        sc.addListener(new RequestContextListener());
    }
}
```

## 5. Conclusion
This tutorial shows how to intercept web requests using Spring MVC’s _HandlerInterceptor_ in order to manually do session management/timeout.

As usual, all examples and configurations are available here on [GitHub](https://github.com/eugenp/tutorials/tree/master/spring-security-mvc-custom).
