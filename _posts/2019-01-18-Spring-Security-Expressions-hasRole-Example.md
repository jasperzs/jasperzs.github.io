---
layout:     post
title:      Spring Security Expressions - hasRole Example
subtitle:   Understanding hasRole expression
date:       2019-01-18
author:     Mr.Humorous 🥘
header-img: img/springFramework/header.jpg
catalog: true
tags:
    - Spring Framework
    - Spring MVC
---

## 1. Overview
Spring Security provides a large variety of Expressions, using the powerful Spring Expression Language ([SpEL](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/expressions.html)). Most of these security expressions are evaluated against a contextual object – __the currently authenticated principal__.

The evaluation of these expressions is performed by the _SecurityExpressionRoot_ – which provides the basis for both web security as well as method level security.

The ability to use [SpEL](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/expressions.html) expressions as an authorization mechanism was introduced in Spring Security 3.0 and is continued in Spring Security 4.x. For the comprehensive list of expressions in Spring Security, have a look at [this guide](https://www.baeldung.com/spring-security-expressions).

## 2. Web Authorization
Spring Security provides two types of web authorization – securing a __full page based on the URL__ and conditionally showing _parts of a JSP page_ based on security rules.

### 2.1. Full Page Authorization Example
With expressions enabled for the _http_ element, an URL pattern can be secured as follows:
```xml
<http use-expressions = "true">
    <intercept-url pattern="/admin/**" access="hasRole('ROLE_ADMIN')" />
    ...
</http>
```

Using Java configuration:
```java
@Configuration
@EnableWebSecurity
public class SecSecurityConfig extends WebSecurityConfigurerAdapter {
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
          .authorizeRequests()
          .antMatchers("/admin/**").hasRole("ADMIN");
    }
    ...
}
```

Spring Security 4 automatically prefixes any role with ROLE_.

The _hasRole_ expression is used here to check if the currently authenticated principal has the specified authority.

### 2.2. In Page Authorization Example
The second kind of web authorization is conditionally showing some part of a JSP page based on the evaluation of a security expression.

Let’s add the required dependency for the __Spring Security JSP taglib support__ in _pom.xml_:
```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-taglibs</artifactId>
    <version>4.2.1.RELEASE</version>
</dependency>
```

The _taglib_ support has to be enabled on the page in order to use the _security_ namespace:
```xml
<%@ taglib prefix="security"
  uri="http://www.springframework.org/security/tags" %>
```

The _hasRole_ expression can now be used on the page, to show/hide HTML elements based on who is currently authenticated when the page is rendered:
```xml
<security:authorize access="hasRole('ROLE_USER')">
    This text is only visible to a user
    <br/>
</security:authorize>
<security:authorize access="hasRole('ROLE_ADMIN')">
    This text is only visible to an admin
    <br/>
</security:authorize>
```

## 3. Method Level Authorization Example – @PreAuthorize
Security Expressions can be used to secure business functionality at the method level as well, by using annotations.

The older _@Secured_ annotations did not allow expressions to be used. Starting with Spring Security 3, the more flexible annotations _@PreAuthorize_ and _@PostAuthorize_ (as well as _@PreFilter_ and _@PostFilter_) are preferred, as they support Spring Expression Language ([SpEL](http://docs.spring.io/spring/docs/current/spring-framework-reference/html/expressions.html)) and provide expression-based access control.

First, in order to use method level security, it needs to be enabled in the security configuration:
```xml
<global-method-security pre-post-annotations="enabled" />
```

For Java configuration, we need to annotate a _@Configuration_ class with _@EnableGlobalMethodSecurity_:
```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig extends WebSecurityConfigurerAdapter {
    ...
}
```

Then, methods can be secured using the Spring _@PreAuthorize_ annotation:
```java
@Service
public class FooService {
    @PreAuthorize("hasRole('ROLE_ADMIN')")
    public List<Foo> findAll() { ... }
    ...
}
```

Now, only principals with ADMIN role will be able to call the _findAll_ method successfully.

Note that the _Pre_ and _Post_ annotations are evaluated and __enforced via proxies__ – in case CGLIB proxies are used, the class and the public methods must not be declared as _final_.

## 4. Programmatic Checking of the Role
A user authority can also be checked programmatically, in raw java code, if the request object is available:
```java
@RequestMapping
public void someControllerMethod(HttpServletRequest request) {
    request.isUserInRole("someAuthority");
}
```

And of course, without access to the _request_, the check can also be done manually by simply verifying if the currently authenticated user has that particular authority. [The user can be obtained](https://www.baeldung.com/get-user-in-spring-security) from the Spring Security context in a variety of ways.
