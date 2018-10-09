---
layout:     post
title:      Intro to Spring Security Expressions
subtitle:   Basic usages of Spring Security Expressions
date:       2019-01-20
author:     Mr.Humorous 🥘
header-img: img/springFramework/header.jpg
catalog: true
tags:
    - Spring Framework
    - Spring MVC
---

## 1. Maven Dependencies
```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-web</artifactId>
        <version>4.1.1.RELEASE</version>
    </dependency>
</dependencies>
```

This dependency only covers Spring Security; don’t forget to add _spring-core_ and _spring-context_ for a full web application.

## 2. Configuration
JAVA:
```java
@Configuration
@EnableAutoConfiguration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityWithoutCsrfConfig extends WebSecurityConfigurerAdapter {
    ...
}
```

XML:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans ...>
    <global-method-security pre-post-annotations="enabled"/>
</beans:beans>
```

## 3. Web Security Expressions
- hasRole, hasAnyRole
- hasAuthority, hasAnyAuthority
- permitAll, denyAll
- isAnonymous, isRememberMe, isAuthenticated, isFullyAuthenticated
- principal, authentication
- hasPermission

## 3.1 hasRole, hasAnyRole
These expressions are responsible for defining the access control or authorization to specific URLs or methods in your application.
```java
@Override
protected void configure(final HttpSecurity http) throws Exception {
    ...
    .antMatchers("/auth/admin/*").hasRole("ADMIN")
    .antMatchers("/auth/*").hasAnyRole("ADMIN","USER")
    ...
}
```

In this example we specify access to all links starting with _/auth/_ restricted to users that are logged in with role _USER_ or role _ADMIN_. Moreover, to access links starting with _/auth/admin/_ we need to have _ADMIN_ role in the system.

The same configuration might be achieved in XML file, by writing:
```xml
<http>
    <intercept-url pattern="/auth/admin/*" access="hasRole('ADMIN')"/>
    <intercept-url pattern="/auth/*" access="hasAnyRole('ADMIN','USER')"/>
</http>
```

## 3.2 hasAuthority, hasAnyAuthority
Roles and authorities are similar in Spring.

The main difference is that, roles have special semantics – starting with Spring Security 4, the *‘ROLE_‘* prefix is automatically added (if it’s not already there) by any role related method.

So _hasAuthority(‘ROLE_ADMIN’)_ is similar to _hasRole(‘ADMIN’)_ because the *‘ROLE_‘* prefix gets added automatically.

__But the good thing about using authorities is that we don’t have to use the ROLE_ prefix at all.__

Here’s a quick example where we’re defining users with specific authorities:
```java
@Override
protected void configure(AuthenticationManagerBuilder auth) throws Exception {
    auth.inMemoryAuthentication()
      .withUser("user1").password("user1Pass")
      .authorities("USER")
      .and().withUser("admin").password("adminPass")
      .authorities("ADMIN");
}
```

We can then of course use these authorities expressions:
```java
@Override
protected void configure(final HttpSecurity http) throws Exception {
    ...
    .antMatchers("/auth/admin/*").hasAuthority("ADMIN")
    .antMatchers("/auth/*").hasAnyAuthority("ADMIN", "USER")
    ...
}
```

As we can see – we’re not mentioning roles at all here.

Finally – we can of course achieve the same functionality using XML configuration as well:
```xml
<authentication-manager>
    <authentication-provider>
        <user-service>
            <user name="user1" password="user1Pass" authorities="ROLE_USER"/>
            <user name="admin" password="adminPass" authorities="ROLE_ADMIN"/>
        </user-service>
    </authentication-provider>
</authentication-manager>

<http>
    <intercept-url pattern="/auth/admin/*" access="hasAuthority('ADMIN')"/>
    <intercept-url pattern="/auth/*" access="hasAnyAuthority('ADMIN','USER')"/>
</http>
```

## 3.3 permitAll, denyAll
Those two annotations are also quite straightforward. We either may permit access to some URL in our service or we may deny access.
```java
...
.antMatchers("/*").permitAll()
...
```

With this config, we will authorize all users (both anonymous and logged in) to access the page starting with ‘/’ (for example, our homepage).

We can also deny access to our entire URL space:
```java
...
.antMatchers("/*").denyAll()
...
```

And again, the same config can be done with an XML config as well:
```xml
<http auto-config="true" use-expressions="true">
    <intercept-url access="permitAll" pattern="/*" /> <!-- Choose only one -->
    <intercept-url access="denyAll" pattern="/*" /> <!-- Choose only one -->
</http>
```

## 3.4 isAnonymous, isRememberMe, isAuthenticated, isFullyAuthenticated
Let’s start with user that didn’t log in to our page. By specifying following in Java config, we enable all unauthorized users to access our main page:
```java
...
.antMatchers("/*").anonymous()
...
```

The same in XML config:
```xml
<http>
    <intercept-url pattern="/*" access="isAnonymous()"/>
</http>
```

If we want to secure the website that everybody who uses it will be required to login, we need to use isAuthenticated() method:
```java
...
.antMatchers("/*").authenticated()
...
```

or XML version:
```xml
<http>
    <intercept-url pattern="/*" access="isAuthenticated()"/>
</http>
```

Moreover, we have two additional expressions, _isRememberMe()_ and _isFullyAuthenticated()_. Through the use of cookies, Spring enables remember-me capabilities so there is no need to log into the system each time. You can read [more about Remember Me here](https://www.baeldung.com/spring-security-remember-me).

In order to give the access to users that were logged in only by remember me function, we may use this:
```java
...
.antMatchers("/*").rememberMe()
...
```

or XML version:
```xml
<http>
    <intercept-url pattern="/*" access="isRememberMe()"/>
</http>
```

Finally, some parts of our services require the user to be authenticated again even if the user is already logged in. For example, user wants to change settings or payment information; it’s of course good practice to ask for manual authentication in the more sensitive areas of the system.

In order to do that, we may specify _isFullyAuthenticated()_, which returns _true_ if the user is not an anonymous or a remember-me user:
```java
...
.antMatchers("/*").fullyAuthenticated()
...
```

or the XML version:
```xml
<http>
    <intercept-url pattern="/*" access="isFullyAuthenticated()"/>
</http>
```

## 3.5 principal, authentication
These expressions allow accessing the _principal_ object representing the current authorized (or anonymous) user and the current _Authentication_ object from the SecurityContext, respectively.

We can, for example, use _principal_ to load a user’s email, avatar, or any other data that is accessible in the logged in user.

And _authentication_ provides information about the full _Authentication_ object, along with its granted authorities.

Both are described in further detail in the following article: [Retrieve User Information in Spring Security](https://www.baeldung.com/get-user-in-spring-security).

## 3.6 hasPermission APIs
This expression is [documented](https://docs.spring.io/spring-security/site/docs/current/reference/html/authorization.html#el-access) and intended to bridge between the expression system and Spring Security’s ACL system, allowing us to specify authorization constraints on individual domain objects, based on abstract permissions.

Let’s have a look at an example. We have a service that allows cooperative writing articles, with a main editor, deciding which article proposed by other authors should be published.

In order to allow usage of such a service, we may create following methods with access control methods:
```java
@PreAuthorize("hasPermission(#articleId, 'isEditor')")
public void acceptArticle(Article article) {
   …
}
```

Only authorized user can call this method, and also user needs to have permission _isEditor_ in the service.

We also need to remember to explicitly configure a _PermissionEvaluator_ in our application context:
```xml
<global-method-security pre-post-annotations="enabled">
    <expression-handler ref="expressionHandler"/>
</global-method-security>

<bean id="expressionHandler"
    class="org.springframework.security.access.expression
      .method.DefaultMethodSecurityExpressionHandler">
    <property name="permissionEvaluator" ref="customInterfaceImplementation"/>
</bean>
```

where _customInterfaceImplementation_ will be the class that implements _PermissionEvaluator_.

Of course we can also do this with Java configuration as well:
```java
@Override
protected MethodSecurityExpressionHandler expressionHandler() {
    DefaultMethodSecurityExpressionHandler expressionHandler =
      new DefaultMethodSecurityExpressionHandler();
    expressionHandler.setPermissionEvaluator(new CustomInterfaceImplementation());
    return expressionHandler;
}
```
