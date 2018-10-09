---
layout:     post
title:      Find The Registered Spring Security Filters
subtitle:   Figure out what Spring Security Filters are registered
date:       2019-03-10
author:     Mr.Humorous 🥘
header-img: img/springFramework/header.jpg
catalog: true
tags:
    - Spring Framework
    - Spring MVC
---

## 1. Overview
Spring Security is based on a chain of servlet filters. Each filter has a specific responsibility and depending on the configuration, filters are added or removed.

In this tutorial, __we’ll discuss different ways to find the registered Spring Security Filters.__

## 2. Security Debugging
First, we’ll enable security debugging which will log detailed security information on each request.

We can enable security debugging using the _debug_ property:
```java
@EnableWebSecurity(debug = true)
```

This way, when we send a request to the server, all the request information will be logged.

We’ll also be able to see the entire security filter chain:
```
Security filter chain: [
  WebAsyncManagerIntegrationFilter
  SecurityContextPersistenceFilter
  HeaderWriterFilter
  LogoutFilter
  UsernamePasswordAuthenticationFilter
  // ...
]
```

## 3. Logging
Next, we’ll find our security filters by enabling the logging for the _FilterChainProxy_.

We can enable logging by adding the following line to _application.properties_:
```
logging.level.org.springframework.security.web.FilterChainProxy=DEBUG
```

Here’s the related log:
```
DEBUG o.s.security.web.FilterChainProxy - /foos/1 at position 1 of 12 in additional filter chain; firing Filter: 'WebAsyncManagerIntegrationFilter'
DEBUG o.s.security.web.FilterChainProxy - /foos/1 at position 2 of 12 in additional filter chain; firing Filter: 'SecurityContextPersistenceFilter'
DEBUG o.s.security.web.FilterChainProxy - /foos/1 at position 3 of 12 in additional filter chain; firing Filter: 'HeaderWriterFilter'
DEBUG o.s.security.web.FilterChainProxy - /foos/1 at position 4 of 12 in additional filter chain; firing Filter: 'LogoutFilter'
DEBUG o.s.security.web.FilterChainProxy - /foos/1 at position 5 of 12 in additional filter chain; firing Filter: 'UsernamePasswordAuthenticationFilter'
...
```

## 4. Obtaining the Filters Programmatically
Now, we’ll see how to obtain the registered security filters programmatically.

__We’ll use _FilterChainProxy_ to get the security filters.__

First, let’s autowire the _springSecurityFilterChain_ bean:
```java
@Autowired
@Qualifier("springSecurityFilterChain")
private Filter springSecurityFilterChain;
```

Here, we used a _@Qualifier_ with the name _springSecurityFilterChain_ with type _Filter_ instead of _FilterChainProxy_. This is because the method of _springSecurityFilterChain()_ in _[WebSecurityConfiguration](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/config/annotation/web/configuration/WebSecurityConfiguration.html#springSecurityFilterChain--)_, which creates the Spring Security filter chain, return type _Filter_ and not _FilterChainProxy_.

Next, we’ll cast this object to _FilterChainProxy_ and call the _getFilterChains()_ method:
```java
public void getFilters() {
    FilterChainProxy filterChainProxy = (FilterChainProxy) springSecurityFilterChain;
    List<SecurityFilterChain> list = filterChainProxy.getFilterChains();
    list.stream()
      .flatMap(chain -> chain.getFilters().stream())
      .forEach(filter -> System.out.println(filter.getClass()));
}
```

And here’s a sample output:
```
class org.springframework.security.web.context.request.async.WebAsyncManagerIntegrationFilter
class org.springframework.security.web.context.SecurityContextPersistenceFilter
class org.springframework.security.web.header.HeaderWriterFilter
class org.springframework.security.web.authentication.logout.LogoutFilter
class org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter
...
```

Note that since Spring Security 3.1, __*FilterChainProxy* is configured using a list of *SecurityFilterChain*__. However, most applications need only one _SecurityFilterChain_.

## 5. Important Spring Security Filters
Finally, let’s take a look at some of the important security filters:
- _UsernamePasswordAuthenticationFilter_: process authentication, responds by default to “/login” URL
- _AnonymousAuthenticationFilter_: when there’s no authentication object in SecurityContextHolder, it creates an anonymous authentication object and put it there
- _FilterSecurityInterceptor_: raise exceptions when access is denied
- _ExceptionTranslationFilter_: catch Spring Security exceptions

## 6. Conclusion
In this quick articles, we explored how to find the registered Spring Security filters programmatically and using logs.

As always, source code can be found over on [GitHub](https://github.com/eugenp/tutorials/tree/master/spring-security-mvc-boot).
