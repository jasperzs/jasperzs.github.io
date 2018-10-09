---
layout:     post
title:      Injecting Prototype Bean into a Singleton Instance in Spring
subtitle:   How to inject prototype bean into singleton bean
date:       2018-12-21
author:     Mr.Humorous 🥘
header-img: img/springFramework/header.jpg
catalog: true
tags:
    - Spring Framework
---

## 1. Overview

By default, Spring beans are singletons. The problem arises when we try to wire beans of different scopes. For example, a prototype bean into a singleton. This is known as the scoped bean injection problem.

Example code is at [here](https://github.com/eugenp/tutorials/tree/master/spring-core).

## 2. Prototype Bean Injection Problem

In order to describe the problem, let’s configure the following beans:
```java
@Configuration
public class AppConfig {

    @Bean
    @Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
    public PrototypeBean prototypeBean() {
        return new PrototypeBean();
    }

    @Bean
    public SingletonBean singletonBean() {
        return new SingletonBean();
    }
}
```

Notice that the first bean has a prototype scope, the other one is a singleton.

Now, let’s inject the prototype-scoped bean into the singleton – and then expose it via the _getPrototypeBean()_ method:
```java
public class SingletonBean {

    // ..

    @Autowired
    private PrototypeBean prototypeBean;

    public SingletonBean() {
        logger.info("Singleton instance created");
    }

    public PrototypeBean getPrototypeBean() {
        logger.info(String.valueOf(LocalTime.now()));
        return prototypeBean;
    }
}
```

Then, let’s load up the _ApplicationContext_ and get the singleton bean twice:
```java
public static void main(String[] args) throws InterruptedException {
    AnnotationConfigApplicationContext context
      = new AnnotationConfigApplicationContext(AppConfig.class);

    SingletonBean firstSingleton = context.getBean(SingletonBean.class);
    PrototypeBean firstPrototype = firstSingleton.getPrototypeBean();

    // get singleton bean instance one more time
    SingletonBean secondSingleton = context.getBean(SingletonBean.class);
    PrototypeBean secondPrototype = secondSingleton.getPrototypeBean();

    isTrue(firstPrototype.equals(secondPrototype), "The same instance should be returned");
}
```

Here’s the output from the console:
```java
Singleton Bean created
Prototype Bean created
11:06:57.894
// should create another prototype bean instance here
11:06:58.895
```

__Both beans were initialized only once__, at the startup of the application context.

## 3. Injecting ApplicationContext

We can also inject the _ApplicationContext_ directly into a bean.

__To achieve this, either use the _@Autowire_ annotation or implement the _ApplicationContextAware_ interface__:
```java
public class SingletonAppContextBean implements ApplicationContextAware {

    private ApplicationContext applicationContext;

    public PrototypeBean getPrototypeBean() {
        return applicationContext.getBean(PrototypeBean.class);
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext)
      throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

Every time the _getPrototypeBean()_ method is called, a new instance of _PrototypeBean_ will be returned from the _ApplicationContext_.

__However, this approach has serious disadvantages__. It contradicts the principle of inversion of control, as we request the dependencies from the container directly.

Also, we fetch the prototype bean from the _applicationContext_ within the _SingletonAppcontextBean_ class. __This means coupling the code to the Spring Framework__.

## 4. Method Injection

Another way to solve the problem is method injection with the **_@Lookup_ annotation**:
```java
@Component
public class SingletonLookupBean {

    @Lookup
    public PrototypeBean getPrototypeBean() {
        return null;
    }
}
```

Spring will override the _getPrototypeBean()_ method annotated with _@Lookup_. It then registers the bean into the application context. Whenever we request the _getPrototypeBean()_ method, it returns a new _PrototypeBean_ instance.

__It will use CGLIB to generate the bytecode__ responsible for fetching the PrototypeBean from the application context.

## 5. javax.inject API

Here’s the singleton bean:
```java
public class SingletonProviderBean {

    @Autowired
    private Provider<PrototypeBean> myPrototypeBeanProvider;

    public PrototypeBean getPrototypeInstance() {
        return myPrototypeBeanProvider.get();
    }
}
```

We use _Provider_ interface to inject the prototype bean. For each _getPrototypeInstance()_ method call, the _myPrototypeBeanProvider.get()_ method returns a new instance of _PrototypeBean_.

## 6. Scoped Proxy

By default, Spring holds a reference to the real object to perform the injection. __Here, we create a proxy object to wire the real object with the dependent one__.

Each time the method on the proxy object is called, the proxy decides itself whether to create a new instance of the real object or reuse the existing one.

To set up this, we modify the _Appconfig_ class to add a new _@Scope_ annotation:
```java
@Scope(
  value = ConfigurableBeanFactory.SCOPE_PROTOTYPE,
  proxyMode = ScopedProxyMode.TARGET_CLASS)
  ```

By default, Spring uses CGLIB library to directly subclass the objects. To avoid CGLIB usage, we can configure the proxy mode with _ScopedProxyMode_.INTERFACES, to use the JDK dynamic proxy instead.

## 7. ObjectFactory Interface

Spring provides the ObjectFactory<T> interface to produce on demand objects of the given type:
```java
public class SingletonObjectFactoryBean {

    @Autowired
    private ObjectFactory<PrototypeBean> prototypeBeanObjectFactory;

    public PrototypeBean getPrototypeInstance() {
        return prototypeBeanObjectFactory.getObject();
    }
}
```

Let’s have a look at _getPrototypeInstance()_ method; _getObject()_ returns a brand new instance of _PrototypeBean_ for each request. Here, we have more control over initialization of the prototype.

Also, the _ObjectFactory_ is a part of the framework; this means avoiding additional setup in order to use this option.

## 8. Create a Bean at Runtime Using java.util.Function

nother option is to create the prototype bean instances at runtime, which also allows us to add parameters to the instances.

To see an example of this, let’s add a _name_ field to our _PrototypeBean_ class:
```java
public class PrototypeBean {
    private String name;

    public PrototypeBean(String name) {
        this.name = name;
        logger.info("Prototype instance " + name + " created");
    }

    //...
}
```

Next, we’ll inject a bean factory into our singleton bean by making use of the _java.util.Function_ interface:
```java
public class SingletonFunctionBean {

    @Autowired
    private Function<String, PrototypeBean> beanFactory;

    public PrototypeBean getPrototypeInstance(String name) {
        PrototypeBean bean = beanFactory.apply(name);
        return bean;
    }

}
```

Finally, we have to define the factory bean, prototype and singleton beans in our configuration:
```java
@Configuration
public class AppConfig {
    @Bean
    public Function<String, PrototypeBean> beanFactory() {
        return name -> prototypeBeanWithParam(name);
    }

    @Bean
    @Scope(value = "prototype")
    public PrototypeBean prototypeBeanWithParam(String name) {
       return new PrototypeBean(name);
    }

    @Bean
    public SingletonFunctionBean singletonFunctionBean() {
        return new SingletonFunctionBean();
    }
    //...
}
```

## 9. Testing

Let’s now write a simple JUnit test to exercise the case with _ObjectFactory_ interface:
```java
@Test
public void givenPrototypeInjection_WhenObjectFactory_ThenNewInstanceReturn() {

    AbstractApplicationContext context
     = new AnnotationConfigApplicationContext(AppConfig.class);

    SingletonObjectFactoryBean firstContext
     = context.getBean(SingletonObjectFactoryBean.class);
    SingletonObjectFactoryBean secondContext
     = context.getBean(SingletonObjectFactoryBean.class);

    PrototypeBean firstInstance = firstContext.getPrototypeInstance();
    PrototypeBean secondInstance = secondContext.getPrototypeInstance();

    assertTrue("New instance expected", firstInstance != secondInstance);
}
```

After successfully launching the test, we can see that each time _getPrototypeInstance()_ method called, a new prototype bean instance created.
