---
layout:     post
title:      How are threads allocated to handle servlet request?
subtitle:   Understanding servlet request threads
date:       2019-01-17
author:     Mr.Humorous 🥘
header-img: img/webApp/header.jpg
catalog: true
tags:
    - WebApp
    - Tomcat
---

Per request means when an HTTP request is made, a thread is created or retrieved from a pool to serve it. One thread serves the whole request. Thread per connection would be the same thing except the thread is used for an entire connection, which could be multiple requests and could also have a lot of dead time in between requests. Servlet containers are thread per request. There may be some implementations that offer thread per connection, but I don't know, and it seems like it would be very wasteful.

Creating a thread inside another thread doesn't establish any special relationship, and the whole point of doing so in most cases is to let the one thread do more work or terminate while the other thread continues working. Using a different thread to do work required by a request will allow the response to be sent immediately. The thread used to serve that request will also be immediately available for another request, regardless of how long your other thread takes to complete. This is pretty much the way of doing asynchronous work in a thread-per-request servlet container.

<u>Caveat</u>: If you're in a full Java EE container, threads may be managed for you in a way that makes it a bad idea to spawn your own. In that case, you're better off asking the container for a thread, but the general principles are the same.
