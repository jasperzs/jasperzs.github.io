---
layout:     post
title:      Transactions with Spring and JPA
subtitle:   Introduction to Spring Transactions
date:       2019-03-14
author:     Mr.Humorous 🥘
header-img: img/springFramework/header.jpg
catalog: true
tags:
    - Spring Framework
    - Spring Data
---

## 1. Overview
This tutorial will discuss __the right way to configure Spring Transactions__, how to use the _@Transactional_ annotation and common pitfalls.

For a more in-depth discussion on the core persistence configuration, check out the [Spring with JPA tutorial](https://www.baeldung.com/the-persistence-layer-with-spring-and-jpa).

There are two distinct ways to configure Transactions – __annotations and AOP__ – each with their own advantages – we’re going to discuss the more common __annotation-config__ here.

## 2. Configure Transactions without XML
Spring 3.1 introduces the __@EnableTransactionManagement__ annotation to be used in on _@Configuration_ classes and enable transactional support:
```java
@Configuration
@EnableTransactionManagement
public class PersistenceJPAConfig{
 
   @Bean
   public LocalContainerEntityManagerFactoryBean
     entityManagerFactoryBean(){
      //...
   }
 
   @Bean
   public PlatformTransactionManager transactionManager(){
      JpaTransactionManager transactionManager
        = new JpaTransactionManager();
      transactionManager.setEntityManagerFactory(
        entityManagerFactoryBean().getObject() );
      return transactionManager;
   }
}
```

## 3. Configure Transactions with XML
Before 3.1 or if Java is not an option, here is the __XML configuration__, using _annotation-driven_ and the namespace support:
```xml
<bean id="txManager" class="org.springframework.orm.jpa.JpaTransactionManager">
   <property name="entityManagerFactory" ref="myEmf" />
</bean>
<tx:annotation-driven transaction-manager="txManager" />
```

## 4. The @Transactional Annotation
With transactions configured, a bean can now be annotated with @Transactional either at the class or method level:
```java
@Service
@Transactional
public class FooService {
    //...
}
```

The annotation supports __further configuration__ as well:
- the __Propagation Type__ of the transaction
- the __Isolation Level__ of the transaction
- a __Timeout__ for the operation wrapped by the transaction
- a __readOnly flag__ – a hint for the persistence provider that the transaction should be read only
- the __Rollback__ rules for the transaction

Note that – by default, rollback happens for runtime, unchecked exceptions only. __The checked exception does not trigger a rollback__ of the transaction; the behavior can, of course, be configured with the _rollbackFor_ and _noRollbackFor_ annotation parameters.

## 5. Potential Pitfalls

### 5.1. Transactions and Proxies
At a high level, __Spring creates proxies for all the classes annotated with *@Transactional*__ – either on the class or on any of the methods. The proxy allows the framework to inject transactional logic before and after the method being invoked – mainly for __starting and committing the transaction__.

What is important to keep in mind is that, if the transactional bean is implementing an interface, by default __the proxy will be a Java Dynamic Proxy__. This means that only external method calls that come in through the proxy will be intercepted – __any self-invocation calls will not start any transaction__ – even if the method is annotated with _@Transactional_.

Another caveat of using proxies is that __only public methods should be annotated with *@Transactional*__ – methods of any other visibilities will simply ignore the annotation silently as these are not proxied.

This [article](http://nurkiewicz.blogspot.ro/2011/10/spring-pitfalls-proxying.html) discusses further proxying pitfalls in great detail here.

### 5.2. Changing the Isolation level
You can change transaction isolation level – as follows:
```java
@Transactional(isolation = Isolation.SERIALIZABLE)
```

Note that this has actually been [introduced](https://jira.spring.io/browse/SPR-5012) in Spring 4.1; if we run the above example before Spring 4.1, it will result in:
> org.springframework.transaction.InvalidIsolationLevelException: Standard JPA does not support custom isolation levels – use a special JpaDialect for your JPA implementation

### 5.3. Read-Only Transactions
The __*readOnly* flag__ usually generates confusion, especially when working with JPA; from the Javadoc:
> This just serves as a hint for the actual transaction subsystem; it will not necessarily cause failure of write access attempts. A transaction manager which cannot interpret the read-only hint will not throw an exception when asked for a read-only transaction.

The fact is that it __cannot be guaranteed__ that an insert or update will not occur when the _readOnly_ flag is set – its behavior is __vendor dependent__ whereas JPA is vendor agnostic.

It is also important to understand that the _readOnly_ flag is only relevant __inside a transaction__; if an operation occurs outside of a transactional context, the flag is simply ignored. A simple example of that would call a method annotated with:
```java
@Transactional( propagation = Propagation.SUPPORTS,readOnly = true )
```

from a non-transactional context – a transaction will not be created and the _readOnly_ flag will be ignored.

### 5.4. Transaction Logging
Transactional related issues can also be better understood by __fine-tuning logging__ in the transactional packages; the relevant package in Spring is _“org.springframework.transaction”, which should be configured with a logging level of TRACE_.

## 6. Conclusion
We covered the basic configuration of transactional semantics using both java and XML, how to use _@Transactional_ and best practices of a Transactional Strategy. The Spring support for __transactional testing__ as well as some __common JPA pitfalls__ was also discussed.

As always, the code presented in this article is available over on [Github](https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-jpa). This is a Maven based project, so it should be easy to import and run as it is.
