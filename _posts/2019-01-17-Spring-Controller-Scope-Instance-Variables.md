---
layout:     post
title:      Scope of a Spring-Controller and its instance-variables
subtitle:   Understanding Spring controller scope and its instace variables sharing
date:       2019-01-17
author:     Mr.Humorous 🥘
header-img: img/springFramework/header.jpg
catalog: true
tags:
    - Spring Framework
    - Spring MVC
---

Spring MVC controllers are singletons by default. An object field will be shared and visible for all requests and all sessions forever.

However without any synchronization you might run into all sorts of concurrency issues (race conditions, visibility). Thus your field should have `volatile` (and `private`, by the way) modifier to avoid visibility issues.

In Spring you can use request- (see [4.5.4.2 Request scope](http://static.springsource.org/spring/docs/current/spring-framework-reference/html/beans.html#beans-factory-scopes-session)) and session-scoped (see: [4.5.4.3 Session scope](http://static.springsource.org/spring/docs/current/spring-framework-reference/html/beans.html#beans-factory-scopes-session)) beans. You can inject them to controllers and any other beans (even singletons!), but Spring makes sure each request/session has an independent instance.

Only thing to remember when injecting request- and session-scoped beans into singletons is to wrap them in scoped proxy (example taken from [4.5.4.5 Scoped beans as dependencies](http://static.springsource.org/spring/docs/current/spring-framework-reference/html/beans.html#beans-factory-scopes-other-injection)):
```xml
<!-- an HTTP Session-scoped bean exposed as a proxy -->
<bean id="userPreferences" class="com.foo.UserPreferences" scope="session">

    <!-- instructs the container to proxy the surrounding bean -->
    <aop:scoped-proxy/>
</bean>
```
