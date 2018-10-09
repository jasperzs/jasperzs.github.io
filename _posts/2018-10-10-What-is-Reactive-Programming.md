---
layout:     post
title:      Explain Reactive Programming
subtitle:   What is Reactive Programming?
date:       2018-10-10
author:     Mr.Humorous 🥘
header-img: img/reactiveProgramming/header.png
catalog: true
tags:
    - Reactive Programming
---

## 1. What is Reactive Programming?
Reactive programming is a development model structured around asynchronous data streams.

## 2. Why do we need Asynchronous work?
To make application more responsive and deliver a smooth user experience without freezing/slowing down the main thread.

To keep the main thread free we need to do a lot of heavy and time-consuming works in the background.
We also want to do heavy work and complex calculations on our servers as mobile devices are not very powerful to do the heavy lifting.

## 3. What do we need from asynchronous library?
![Characteristics of asynchronous library illustration]({{ site.url }}/img/reactiveProgramming/whatIsReactiveProgramming/EvaluationMatrix.png?raw=true)
+ __Explicit execution__: <ins>If we start the execution of a bunch of work on a new thread, we should be able to control it.</ins>
                          If you are going to perform some background task, you gather the information and prepare them. As soon as you are ready, you can kick-off the background task.
+ __Easy thread management__: <ins>In asynchronous work, thread management is the key.</ins>
                              We often need to update the UI on the main thread from the background thread in the middle of the task or at the end of the task.
                              For that, we need to pass our work from one thread (background thread) to another thread (here main thread).
                              So you should be able to switch the thread easily and pass the work to another thread when needed.
+ __Easily composable__: Ideally, It would be great if we can create an asynchronous work and as we start spinning background thread, it just do it’s work without depending any other thread (especially on UI thread).
                         And it also stays independent from the other thread until it finishes its job. But in the real world, we need to update the UI, make database changes and many more things.
                         All these make threading interdependent. So the asynchronous library should be <ins>easily composable and provide less room for the error.</ins>
+ __Minimum the side effects__: While working with multiple threads, <ins>the other thread should experience minimum side effects from the other thread.</ins>
                            That makes your code easily readable and understandable to a new person and it also makes error easily traceable.

## 4. Reactive Extension (RX) - An implementation of Reactive Programming
RX = Observable + Observer + Schedulers
+ __Observable__: Data streams which pack the data that can be passed around from one thread to another thread. They emit the data periodically or only once in their life cycle.
+ __Observer__: They consume the data stream emitted by the observable. Here they can perform various operations like parsing the Json response or updating the UI.
+ __Scheduler__: Thread management components that tell Observable and Observer on which thread they should run.

### Example using RxJava
![How to use RxJava Illustration]({{ site.url }}/img/reactiveProgramming/whatIsReactiveProgramming/howToUseRx.png)
```java
Observable<String> database = Observable      //Observable. This will emit the data
                .just(new String[]{"1", "2", "3", "4"});    //Operator

Observer<String> observer = new Observer<String>() {
           @Override
            public void onCompleted() {
                //...
            }

            @Override
            public void onError(Throwable e) {
                //...
            }

            @Override
            public void onNext(String s) {
                //...
            }
        };

database.subscribeOn(Schedulers.newThread())          //Observable runs on new background thread.
        .observeOn(AndroidSchedulers.mainThread())    //Observer will run on main UI thread.
        .subscribe(observer);                         //Subscribe the observer
```
