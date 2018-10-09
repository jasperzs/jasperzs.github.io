---
layout:     post
title:      What is Spring Security Principal?
subtitle:   Understanding Spring Security Principal/User
date:       2019-01-17
author:     Mr.Humorous 🥘
header-img: img/springFramework/header.jpg
catalog: true
tags:
    - Spring Framework
    - Spring MVC
---

The <u>principal</u> is the currently logged in user. However, you retrieve it through the security context which is bound to the current thread and as such it's also bound to the current request and its session.

`SecurityContextHolder.getContext()` internally obtains the current `SecurityContext` implementation through a `ThreadLocal` variable. Because a request is bound to a single thread this will get you the context of the current request.

To simplify you could say that the security context is in the session and contains user/principal and roles/authorities.

## How do I retrieve a specific user?

You don't. All APIs are designed to allow access to the user & session of the current request. Let user A be one of 100 currently authenticated users. If A issues a request against your server it will allocate one thread to process that request. If you then do `SecurityContextHolder.getContext().getAuthentication()` you do so in the context of this thread. By default from within that thread you don't have access to the context of user B which is processed by a different thread.

## And how do I differentiate between users that are doing requests?
You don't have to, that's what the Servlet container does for you.
