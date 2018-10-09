---
layout:     post
title:      A Custom Filter In The Spring Security Filter Chain
subtitle:   Create a custom security filter
date:       2019-03-10
author:     Mr.Humorous 🥘
header-img: img/springFramework/header.jpg
catalog: true
tags:
    - Spring Framework
    - Spring MVC
---

## 1. Overview
In this quick article, we’ll focus on writing a custom filter for the Spring Security filter chain.

## 2. Creating the Filter
Spring Security provides a number of filters by default, and most of the time, these are enough.

__But of course sometimes it’s necessary to implement new functionality with create a new filter to use in the chain.__

We’ll start by implementing the [org.springframework.web.filter.GenericFilterBean](http://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/filter/GenericFilterBean.html).

The _GenericFilterBean_ is a simple [javax.servlet.Filter](http://docs.oracle.com/javaee/7/api/javax/servlet/Filter.html) implementation that is Spring aware.

On to the implementation – we only need to implement a single method:
```java
public class CustomFilter extends GenericFilterBean {

    @Override
    public void doFilter(
      ServletRequest request,
      ServletResponse response,
      FilterChain chain) throws IOException, ServletException {
        chain.doFilter(request, response);
    }
}
```

## 3. Using the Filter in the Security Config
We’re free to choose either XML configuration or Java configuration to wire the filter into the Spring Security configuration.

### 3.1. Java Configuration
You can register the filter programmatically overriding the _[configure](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/config/annotation/web/configuration/WebSecurityConfigurerAdapter.html#configure-org.springframework.security.config.annotation.web.builders.HttpSecurity-)_ method from _[WebSecurityConfigurerAdapter](https://docs.spring.io/spring-security/site/docs/current/api/org/springframework/security/config/annotation/web/configuration/WebSecurityConfigurerAdapter.html)_. For example, it works with the addFilterAfter method on a _[HttpSecurity](https://docs.spring.io/spring-security/site/docs/5.0.0.M5/api/org/springframework/security/config/annotation/web/builders/HttpSecurity.html)_ instance:
```java
@Configuration
public class CustomWebSecurityConfigurerAdapter
  extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.addFilterAfter(
          new CustomFilter(), BasicAuthenticationFilter.class);
    }
}
```

There are a couple of possible methods:
- _[addFilterBefore(filter, class)](https://docs.spring.io/spring-security/site/docs/5.0.0.M5/api/org/springframework/security/config/annotation/web/builders/HttpSecurity.html#addFilterBefore(javax.servlet.Filter,%20java.lang.Class))_ – adds a filter before the position of the specified filter class
- _[addFilterAfter(filter, class)](https://docs.spring.io/spring-security/site/docs/5.0.0.M5/api/org/springframework/security/config/annotation/web/builders/HttpSecurity.html#addFilterAfter(javax.servlet.Filter,%20java.lang.Class))_ – adds a filter after the position of the specified filter class
- _[addFilterAt(filter, class)](https://docs.spring.io/spring-security/site/docs/5.0.0.M5/api/org/springframework/security/config/annotation/web/builders/HttpSecurity.html)_ – adds a filter at the location of the specified filter class
- _[addFilter(filter)](https://docs.spring.io/spring-security/site/docs/5.0.0.M5/api/org/springframework/security/config/annotation/web/builders/HttpSecurity.html#addFilter(javax.servlet.Filter))_ – adds a filter that must be an instance of or extend one of the filters provided by Spring Security

### 3.2. XML Configuration
You can add the filter to the chain using the _custom-filter_ tag and one of [these names](http://docs.spring.io/autorepo/docs/spring-security/current/reference/htmlsingle/#filter-stack) to specify the position of your filter. For instance, it can be pointed out by the _after_ attribute:
```xml
<http>
    <custom-filter after="BASIC_AUTH_FILTER" ref="myFilter" />
</http>

<beans:bean id="myFilter" class="org.baeldung.security.filter.CustomFilter"/>
```

Here are all attributes to specify exactly a place your filter in the stack:
- _after_ – describes the filter immediately after which a custom filter will be placed in the chain
- _before_ – defines the filter before which our filter should be placed in the chain
- _position_ – allows replacing a standard filter in the explicit position by a custom filter

## 4. Conclusion
In this quick article, we created a custom filter and wired that into the Spring Security filter chain.

As always, all code examples are available in the sample [Github](https://github.com/eugenp/tutorials/tree/master/spring-security-rest-basic-auth) project.
