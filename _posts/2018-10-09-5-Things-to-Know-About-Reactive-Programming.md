---
layout:     post
title:      5 Things to Know About Reactive Programming
subtitle:   Memorize the following 5 things
date:       2018-10-09
author:     Mr.Humorous 🥘
header-img: img/reactiveProgramming/header.png
catalog: true
tags:
    - Reactive Programming
---

## 1. Reactive Programming Is Programming With Asynchronous Data Streams
Events, messages, calls, and even failures are going to be conveyed by a data stream. You observe these streams and react when value is emitted.

In your code, you are going to create data streams of anything and from anything: click events, HTTP requests, ingested messages, availability notifications, changes on a variable, cache events, measures from a sensor, literally anything that may change or happen. Your application becomes inherently asynchronous.

![Reactive Programming Illustration]({{ site.url }}/img/reactiveProgramming/5ThingsToKnow/illustration.png?raw=true)

__RX (Reactive eXtension)__ is an implementation of the reactive programming principles. With it, your code creates and subscribes to data streams named <ins>_Observable_</ins>

![Reactive Extension Illustration]({{ site.url }}/img/reactiveProgramming/5ThingsToKnow/reactiveExtension.png?raw=true)

## 2. Observables Can Be Cold or Hot

### 2.1 Cold Observable
Cold observables are lazy. They don’t do anything until someone starts observing them (subscribe in RX). They only start running when they are consumed.
The data produced by a cold stream is not shared among subscribers and when you subscribe you get all the items.

### 2.2 Hot Observable
Hot streams are active before the subscription. The data is independent of an individual subscriber.
When an observer subscribes to a hot observable, it will get all values in the stream that are emitted after it subscribes. The values are shared among all subscribers.
If you are not subscribed to a hot observable, you won’t receive the data, and this data is lost.

## 3. Misused Asynchrony Bites
By structuring your program around data streams, you are writing code invoked when the stream emits a new item.
Threads, blocking code and side-effects are then very important matters.

### 3.1 Side Effects
Functions without side-effects interact with the rest of the program exclusively through their arguments and return values (Imagine Angular NgRx Effects).
In all cases, you need to avoid unnecessary side-effects as they affect thread safety.

### 3.2 Threads
You must never forget forget who is calling you, or on which thread your functions are executed. It is heavily recommended to avoid using too many threads in your program.

### 3.3 Never Block
Because you don’t own the thread calling you, you must be sure to never block it. If you do you may avoid the other items to be emitted, they will be buffered until the buffer is full.
```
client.get("/api/people/4")
.rxSend()
.map(HttpResponse::bodyAsJsonObject)
.map(json -> json.getString("name"))
.subscribe(System.out::println, Throwable::printStackTrace);
```

## 4. Keep Things Simple
```java
manager.getCampaignById(id)
  .flatMap(campaign ->
    manager.getCartsForCampaign(campaign)
      .flatMap(list -> {
        Single<List<Product>> products = manager.getProducts(campaign);
        Single<List<UserCommand>> carts = manager.getCarts(campaign);
        return products.zipWith(carts,
            (p, c) -> new CampaignModel(campaign, p, c));
      })
     .flatMap(model -> template
        .rxRender(rc, "templates/fruits/campaign.thl.html")
        .map(Buffer::toString))
    )
    .subscribe(
      content -> rc.response().end(content),
     err -> {
      log.error("Unable to render campaign view", err);
      getAllCampaigns(rc);
    }
);
```
Bad code snippet like this chains several asynchronous operations (flatmap), join another set of operations (zip).
Don’t abuse, write comments, explain, or draw diagrams

## 5. Reactive Programming != Reactive System
Using __reactive programming does not build a reactive system__. Reactive systems are an architectural style to _build responsive distributed systems_.
+ __Responsive__: a reactive system needs to handle requests in a reasonable time.
+ __Resilient__: a reactive system must stay responsive in the face of failures (crash, timeout, 500 errors… ), so it must be designed for failures and deal with them appropriately.
+ __Elastic__: a reactive system must stay responsive under various loads. Consequently, it must scale up and down, and be able to handle the load with minimal resources.
+ __Message driven__: components from a reactive system interacts using asynchronous message passing.

