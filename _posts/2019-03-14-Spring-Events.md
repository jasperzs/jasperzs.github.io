---
layout:     post
title:      Spring Events
subtitle:   Undetstanding Spring Events
date:       2019-03-14
author:     Mr.Humorous 🥘
header-img: img/springFramework/header.jpg
catalog: true
tags:
    - Spring Framework
---

## 1. Overview
Events are one of the more overlooked functionalities in the framework but also one of the more useful. And – like many other things in Spring – event publishing is one of the capabilities provided by _ApplicationContext_.

There are a few simple guidelines to follow:
- __the event should extend *ApplicationEvent*__
- __the publisher should inject an *ApplicationEventPublisher* object__
- __the listener should implement the *ApplicationListener* interface__

## 2. A Custom Event
Spring allows to create and publish custom events which – by default – __are synchronous__. This has a few advantages – such as, for example, the listener being able to participate in the publisher’s transaction context.

### 2.1. A Simple Application Event
Let’s create __a simple event class__ – just a placeholder to store the event data. In this case, the event class holds a String message:
```java
public class CustomSpringEvent extends ApplicationEvent {
    private String message;

    public CustomSpringEvent(Object source, String message) {
        super(source);
        this.message = message;
    }
    public String getMessage() {
        return message;
    }
}
```

### 2.2. A Publisher
Now let’s create __a publisher of that event__. The publisher constructs the event object and publishes it to anyone who’s listening.

To publish the event, the publisher can simply inject the _ApplicationEventPublisher_ and use the _publishEvent()_ API:
```java
@Component
public class CustomSpringEventPublisher {
    @Autowired
    private ApplicationEventPublisher applicationEventPublisher;

    public void doStuffAndPublishAnEvent(final String message) {
        System.out.println("Publishing custom event. ");
        CustomSpringEvent customSpringEvent = new CustomSpringEvent(this, message);
        applicationEventPublisher.publishEvent(customSpringEvent);
    }
}
```

Alternatively, the publisher class can implement the _ApplicationEventPublisherAware_ interface – this will also inject the event publisher on the application start-up. Usually, it’s simpler to just inject the publisher with _@Autowire_.

### 2.3. A Listener
Finally, let’s create the listener.

The only requirement for the listener is to be a bean and implement _ApplicationListener_ interface:
```java
@Component
public class CustomSpringEventListener implements ApplicationListener<CustomSpringEvent> {
    @Override
    public void onApplicationEvent(CustomSpringEvent event) {
        System.out.println("Received spring custom event - " + event.getMessage());
    }
}
```

Notice how our custom listener is parametrized with the generic type of custom event – which makes the _onApplicationEvent()_ method type safe. This also avoids having to check if the object is an instance of a specific event class and casting it.

And, as already discussed – by default __spring events are synchronous__ – the _doStuffAndPublishAnEvent()_ method blocks until all listeners finish processing the event.

## 3. Creating Asynchronous Events
In some cases, publishing events synchronously isn’t really what we’re looking for – __we may need async handling of our events__.

You can turn that on in the configuration by creating an _ApplicationEventMulticaster_ bean with an executor; for our purposes here _SimpleAsyncTaskExecutor_ works well:
```java
@Configuration
public class AsynchronousSpringEventsConfig {
    @Bean(name = "applicationEventMulticaster")
    public ApplicationEventMulticaster simpleApplicationEventMulticaster() {
        SimpleApplicationEventMulticaster eventMulticaster
          = new SimpleApplicationEventMulticaster();

        eventMulticaster.setTaskExecutor(new SimpleAsyncTaskExecutor());
        return eventMulticaster;
    }
}
```

The event, the publisher and the listener implementations remain the same as before – but now, __the listener will asynchronously deal with the event in a separate thread.__

## 4. Existing Framework Events
Spring itself publishes a variety of events out of the box. For example, the _ApplicationContext_ will fire various framework events. E.g. _ContextRefreshedEvent, ContextStartedEvent, RequestHandledEvent_ etc.

These events provide application developers an option to hook into the lifecycle of the application and the context and add in their own custom logic where needed.

Here’s a quick example of a listener listening for context refreshes:
```java
public class ContextRefreshedListener
  implements ApplicationListener<ContextRefreshedEvent> {
    @Override
    public void onApplicationEvent(ContextRefreshedEvent cse) {
        System.out.println("Handling context re-freshed event. ");
    }
}
```

To learn more about existing framework events, have a look at our next [tutorial](https://www.baeldung.com/spring-context-events) here.

## 5. Annotation-Driven Event Listener
Starting with Spring 4.2, an event listener is not required to be a bean implementing the _ApplicationListener_ interface – it can be registered on any _public_ method of a managed bean via the _@EventListener_ annotation:
```java
@Component
public class AnnotationDrivenContextStartedListener {
    // @Async
    @EventListener
    public void handleContextStart(ContextStartedEvent cse) {
        System.out.println("Handling context started event.");
    }
}
```

As before, the method signature declares the event type it consumes. As before, this listener is invoked synchronously. But now making it asynchronous is as simple as adding an _@Async_ annotation (do not forget to [enable Async support](https://www.baeldung.com/spring-async#enable-async-support) in the application).

## 6. Generics Support
It is also possible to dispatch events with generics information in the event type.

### 6.1. A Generic Application Event
__Let’s create a generic event type.__ In our example, the event class holds any content and a _success_ status indicator:
```java
public class GenericSpringEvent<T> {
    private T what;
    protected boolean success;

    public GenericSpringEvent(T what, boolean success) {
        this.what = what;
        this.success = success;
    }
    // ... standard getters
}
```

Notice the difference between _GenericSpringEvent_ and _CustomSpringEvent_. We now have the flexibility to publish any arbitrary event and it’s not required to extend from _ApplicationEvent_ anymore.

### 6.2. A Listener
Now let’s create __a listener of that event__. We could define the listener by implementing the _ApplicationListener_ interface like before:
```java
@Component
public class GenericSpringEventListener implements ApplicationListener<GenericSpringEvent<String>> {
    @Override
    public void onApplicationEvent(@NonNull GenericSpringEvent<String> event) {
        System.out.println("Received spring generic event - " + event.getWhat());
    }
}
```

But unfortunately, this definition requires us to inherit _GenericSpringEvent_ from the _ApplicationEvent_ class. So for this tutorial, let’s make use of an annotation-driven event listener discussed previously.

It is also possible to __make the event listener conditional__ by defining a boolean SpEL expression on the _@EventListener_ annotation. In this case, the event handler will only be invoked for a successful _GenericSpringEvent_ of _String_:
```java
@Component
public class AnnotationDrivenEventListener {
    @EventListener(condition = "#event.success")
    public void handleSuccessful(GenericSpringEvent<String> event) {
        System.out.println("Handling generic event (conditional).");
    }
}
```

The [Spring Expression Language (SpEL)](https://www.baeldung.com/spring-expression-language) is a powerful expression language that’s covered in details in another tutorial.

### 6.3. A Publisher
The event publisher is similar to the one described above. But due to type erasure, we need to publish an event that resolves the generics parameter we would filter on. For example, _class GenericStringSpringEvent extends GenericSpringEvent<String>_.

And there’s __an alternative way of publishing events__. If we return a non-null value from a method annotated with _@EventListener_ as the result, Spring Framework will send that result as a new event for us. Moreover, we can publish multiple new events by returning them in a collection as the result of event processing.

## 7. Transaction Bound Events
This paragraph is about using the _@TransactionalEventListener_ annotation. To learn more about transaction management check out the [Transactions with Spring and JPA](https://www.baeldung.com/transaction-configuration-with-jpa-and-spring) tutorial.

Since Spring 4.2, the framework provides a new _@TransactionalEventListener_ annotation, which is an extension of _@EventListener_, that allows binding the listener of an event to a phase of the transaction. Binding is possible to the following transaction phases:
- _AFTER_COMMIT_ (default) is used to fire the event if the transaction has __completed successfully__
- _AFTER_ROLLBACK_ – if the transaction has __rolled back__
- _AFTER_COMPLETION_ – if the transaction has __completed__ (an alias for _AFTER_COMMIT_ and _AFTER_ROLLBACK_)
- _BEFORE_COMMIT_ is used to fire the event right __before__ transaction __commit__

Here’s a quick example of transactional event listener:
```java
@TransactionalEventListener(phase = TransactionPhase.BEFORE_COMMIT)
public void handleCustom(CustomSpringEvent event) {
    System.out.println("Handling event inside a transaction BEFORE COMMIT.");
}
```

This listener will be invoked only if there’s a transaction in which the event producer is running and it’s about to be committed.

And, if no transaction is running the event isn’t sent at all unless we override this by setting _fallbackExecution_ attribute to _true_.

## 8. Conclusion
In this quick tutorial, we went over the basics of __dealing with events in Spring__ – creating a simple custom event, publishing it, and then handling it in a listener.

We also had a brief look at how to enable asynchronous processing of events in the configuration.

Then we learned about improvements introduced in Spring 4.2, such as annotation-driven listeners, better generics support, and events binding to transaction phases.

As always, the code presented in this article is available over on [Github](https://github.com/eugenp/tutorials/tree/master/spring-all). This is a Maven based project, so it should be easy to import and run as it is.
