---
layout:     post
title:      Spring Bean Scopes
subtitle:   Quick Guide to Spring Bean Scopes
date:       2018-12-20
author:     Mr.Humorous 🥘
header-img: img/springFramework/header.jpg
catalog: true
tags:
    - Spring Framework
    - Spring MVC
---

## 1. Overview

The scope of a bean defines the life cycle and visibility of that bean in the contexts in which it is used.

The latest version of Spring framework defines 6 types of scopes:
- singleton
- prototype
- request
- session
- application
- websocket

The last four scopes mentioned _request, session, application and websocket_ are only available in a web-aware application.

Example code is [here](https://github.com/eugenp/tutorials/tree/master/spring-all).

## 2. Singleton Scope

Defining a bean with _singleton_ scope means the container creates a single instance of that bean, and all requests for that bean name will return the same object, which is cached. Any modifications to the object will be reflected in all references to the bean. This scope is the default value if no other scope is specified.

Let’s create a _Person_ entity to exemplify the concept of scopes:
```java
public class Person {
    private String name;

    // standard constructor, getters and setters
}
```

Afterwards, we define the bean with _singleton_ scope by using the _@Scope_ annotation:
```java
@Bean
@Scope("singleton")
public Person personSingleton() {
    return new Person();
}
```

We can also use a constant instead of the _String_ value in the following manner:
```java
@Scope(value = ConfigurableBeanFactory.SCOPE_SINGLETON)
```

Now we proceed to write a test that shows that two objects referring to the same bean will have the same values, even if only one of them changes their state, as they are both referencing the same bean instance:
```java
private static final String NAME = "John Smith";

@Test
public void givenSingletonScope_whenSetName_thenEqualNames() {
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("scopes.xml");

    Person personSingletonA = (Person) applicationContext.getBean("personSingleton");
    Person personSingletonB = (Person) applicationContext.getBean("personSingleton");

    personSingletonA.setName(NAME);
    Assert.assertEquals(NAME, personSingletonB.getName());

    ((AbstractApplicationContext) applicationContext).close();
}
```

The _scopes.xml_ file in this example should contain the xml definitions of the beans used:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
    http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="personSingleton" class="org.baeldung.scopes.Person" scope="singleton"/>
</beans>
```

## 3. Prototype Scope

A bean with _prototype_ scope will return a different instance every time it is requested from the container. It is defined by setting the value _prototype_ to the _@Scope_ annotation in the bean definition:
```java
@Bean
@Scope("prototype")
public Person personPrototype() {
    return new Person();
}
```

We could also use a constant as we did for the singleton scope:
```java
@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)
```

We will now write a similar test as before that shows two objects requesting the same bean name with scope prototype will have different states, as they are no longer referring to the same bean instance:
```java
private static final String NAME = "John Smith";
private static final String NAME_OTHER = "Anna Jones";

@Test
public void givenPrototypeScope_whenSetNames_thenDifferentNames() {
    ApplicationContext applicationContext = new ClassPathXmlApplicationContext("scopes.xml");

    Person personPrototypeA = (Person) applicationContext.getBean("personPrototype");
    Person personPrototypeB = (Person) applicationContext.getBean("personPrototype");

    personPrototypeA.setName(NAME);
    personPrototypeB.setName(NAME_OTHER);

    Assert.assertEquals(NAME, personPrototypeA.getName());
    Assert.assertEquals(NAME_OTHER, personPrototypeB.getName());

    ((AbstractApplicationContext) applicationContext).close();
}
```

The _scopes.xml_ file is similar to the one presented in the previous section while adding the xml definition for the bean with _prototype_ scope:
```xml
<bean id="personPrototype" class="org.baeldung.scopes.Person" scope="prototype"/>
```

## 4. Web Aware Scopes

As mentioned, there are four additional scopes that are only available in a web-aware application context. These are less often used in practice.

The _request_ scope creates a bean instance for a single HTTP request while _session_ scope creates for an HTTP Session.

The _application_ scope creates the bean instance for the lifecycle a _ServletContext_ and the _websocket_ scope creates it for a particular _WebSocket_ session.

Let’s create a class to use for instantiating the beans:
```java
public class HelloMessageGenerator {
    private String message;

    // standard getter and setter
}
```

### 4.1 Request Scope

We can define the bean with _request_ scope using the _@Scope_ annotation:
```java
@Bean
@Scope(value = WebApplicationContext.SCOPE_REQUEST, proxyMode = ScopedProxyMode.TARGET_CLASS)
public HelloMessageGenerator requestScopedBean() {
    return new HelloMessageGenerator();
}
```

The _proxyMode_ attribute is necessary because, at the moment of the instantiation of the web application context, there is no active request. Spring will create a proxy to be injected as a dependency, and instantiate the target bean when it is needed in a request.

Next, we can define a controller that has an injected reference to the _requestScopedBean_. We need to access the same request twice in order to test the web specific scopes.

If we display the _message_ each time the request is run, we can see that the value is reset to _null_, even though it is later changed in the method. This is because of a different bean instance being returned for each request.
```java
@Controller
public class ScopesController {
    @Resource(name = "requestScopedBean")
    HelloMessageGenerator requestScopedBean;

    @RequestMapping("/scopes/request")
    public String getRequestScopeMessage(final Model model) {
        model.addAttribute("previousMessage", requestScopedBean.getMessage());
        requestScopedBean.setMessage("Good morning!");
        model.addAttribute("currentMessage", requestScopedBean.getMessage());
        return "scopesExample";
    }
}
```

### 4.2. Session Scope

We can define the bean with _session_ scope in a similar manner:
```java
@Bean
@Scope(value = WebApplicationContext.SCOPE_SESSION, proxyMode = ScopedProxyMode.TARGET_CLASS)
public HelloMessageGenerator sessionScopedBean() {
    return new HelloMessageGenerator();
}
```

Next, we define a controller with a reference to the _sessionScopedBean_. Again, we need to run two requests in order to show that the value of the _message_ field is the same for the session.

In this case, when the request is made for the first time, the value message is _null_. But once, it is changed, then that value is retained for subsequent requests as the same instance of the bean is returned for the entire session.
```java
@Controller
public class ScopesController {
    @Resource(name = "sessionScopedBean")
    HelloMessageGenerator sessionScopedBean;

    @RequestMapping("/scopes/session")
    public String getSessionScopeMessage(final Model model) {
        model.addAttribute("previousMessage", sessionScopedBean.getMessage());
        sessionScopedBean.setMessage("Good afternoon!");
        model.addAttribute("currentMessage", sessionScopedBean.getMessage());
        return "scopesExample";
    }
}
```

### 4.3. Application Scope

The _application_ scope creates the bean instance for the lifecycle of a _ServletContext_.

This is similar to the singleton scope but there is a very important difference with regards to the scope of the bean.

When beans are _application_ scoped the same instance of the bean is shared across multiple servlet-based applications running in the same _ServletContext_, while singleton-scoped beans are scoped to a single application context only.

Let’s create the bean with _application_ scope:
```java
@Bean
@Scope(value = WebApplicationContext.SCOPE_APPLICATION, proxyMode = ScopedProxyMode.TARGET_CLASS)
public HelloMessageGenerator applicationScopedBean() {
    return new HelloMessageGenerator();
}
```

And the controller that references this bean:
```java
@Controller
public class ScopesController {
    @Resource(name = "applicationScopedBean")
    HelloMessageGenerator applicationScopedBean;
 
    @RequestMapping("/scopes/application")
    public String getApplicationScopeMessage(final Model model) {
        model.addAttribute("previousMessage", applicationScopedBean.getMessage());
        applicationScopedBean.setMessage("Good afternoon!");
        model.addAttribute("currentMessage", applicationScopedBean.getMessage());
        return "scopesExample";
    }
}
```

In this case, value message once set in the _applicationScopedBean_ will be retained for all subsequent requests, sessions and even for a different servlet application that will access this bean, provided it is running in the same _ServletContext_.

### 4.4. WebSocket Scope

Finally, let’s create the bean with _websocket_ scope:
```java
@Bean
@Scope(scopeName = "websocket", proxyMode = ScopedProxyMode.TARGET_CLASS)
public HelloMessageGenerator websocketScopedBean() {
    return new HelloMessageGenerator();
}
```

WebSocket-scoped beans when first accessed are stored in the _WebSocket_ session attributes. The same instance of the bean is then returned whenever that bean is accessed during the entire _WebSocket_ session.

We can also say that it exhibits singleton behavior but limited to a _WebSocket_ session only.
