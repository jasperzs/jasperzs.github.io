---
layout:     post
title:      Hibernate Entity Lifecycle
subtitle:   Understanding the lifecycle of Hibernate entity
date:       2019-03-16
author:     Mr.Humorous 🥘
header-img: img/hibernate/header.jpeg
catalog: true
tags:
    - Hibernate
---

## 1. Overview
Every Hibernate entity naturally has a lifecycle within the framework – it’s either in a transient, managed, detached or deleted state.

__Understanding these states on both conceptual and technical level is essential to be able to use Hibernate properly.__

To learn about various Hibernate methods that deal with entities, have a look at [one of our previous tutorials](https://www.baeldung.com/hibernate-save-persist-update-merge-saveorupdate).

## 2. Helper Methods
Throughout this tutorial, we’ll consistently use several helper methods:
- _HibernateLifecycleUtil.getManagedEntities(session)_ – we’ll use it to get all managed entities from a Session’s internal store
- _DirtyDataInspector.getDirtyEntities()_ – we’re going to use this method to get a list of all entities that were marked as ‘dirty’
- _HibernateLifecycleUtil.queryCount(query)_ – a convenient method to do count(*) query against the embedded database

All of the above helper methods are statically imported for better readability. You can find their implementations in the GitHub project linked at the end of this article.

## 3. It’s All About Persistence Context
Before getting into the topic of entity lifecycle, first, we need to understand the _persistence context_.

__Simply put, the *persistence context* sits between client code and data store.__ It’s a staging area where persistent data is converted to entities, ready to be read and altered by client code.

Theoretically speaking, the _persistence context_ is an implementation of the [Unit of Work](https://martinfowler.com/eaaCatalog/unitOfWork.html) pattern. It keeps track of all loaded data, tracks changes of that data, and is responsible to eventually synchronize any changes back to the database at the end of the business transaction.

JPA _EntityManager_ and Hibernate’s _Session_ are an implementation of the _persistence context_ concept. Throughout this article, we’ll use Hibernate _Session_ to represent _persistence context_.

Hibernate entity lifecycle state explains how the entity is related to a _persistence context_, as we’ll see next.

## 4. Managed Entity
__A managed entity is a representation of a database table row__ (although that row doesn’t have to exist in the database yet).

This is managed by the currently running _Session_, and __every change made on it will be tracked and propagated to the database automatically__.

The _Session_ either loads the entity from the database or re-attaches a detached entity. We’ll discuss detached entities in section 5.

Let’s observe some code to get clarification.

Our sample application defines one entity, the _FootballPlayer_ class. At startup, we’ll initialize the data store with some sample data:
```
+-------------------+-------+
| Name              |  ID   |
+-------------------+-------+
| Cristiano Ronaldo | 1     |
| Lionel Messi      | 2     |
| Gigi Buffon       | 3     |
+-------------------+-------+
```

Let’s say we want to change the name of Buffon to start with – we want to put in his full name _Gianluigi Buffon_ instead of Gigi Buffon.

First, we need to start our unit of work by obtaining a _Session_:
```java
Session session = sessionFactory.openSession();
```

In a server environment, we may inject a _Session_ to our code via a context-aware proxy. The principle remains the same: we need a _Session_ to encapsulate the business transaction of our unit of work.

Next, we’ll instruct our _Session_ to load the data from the persistent store:
```java
assertThat(getManagedEntities(session)).isEmpty();

List<FootballPlayer> players = s.createQuery("from FootballPlayer").getResultList();

assertThat(getManagedEntities(session)).size().isEqualTo(3);
```

When we first obtain a _Session_, its persistent context store is empty, as shown by our first _assert_ statement.

Next, we’re executing a query which retrieves data from the database, creates entity representation of the data, and finally returns the entity for us to use.

Internally, the _Session_ keeps track of all entities it loads in the persistent context store. In our case, the _Session’s_ internal store will contain 3 entities after the query.

Now let’s change Gigi’s name:
```java
Transaction transaction = session.getTransaction();
transaction.begin();

FootballPlayer gigiBuffon = players.stream()
  .filter(p -> p.getId() == 3)
  .findFirst()
  .get();

gigiBuffon.setName("Gianluigi Buffon");
transaction.commit();

assertThat(getDirtyEntities()).size().isEqualTo(1);
assertThat(getDirtyEntities().get(0).getName()).isEqualTo("Gianluigi Buffon");
```

### 4.1. How Does It Work?
On call to transaction _commit()_ or _flush()_, the _Session_ will find any _dirty_ entities from its tracking list and synchronize the state to the database.

__Notice that we didn’t need to call any method to notify *Session* that we changed something in our entity – since it’s a managed entity, all changes are propagated to the database automatically.__

A managed entity is always a persistent entity – it must have a database identifier, even though the database row representation is not yet created i.e. the INSERT statement is pending the end of the unit of work.

See the chapter about transient entities below.

## 5. Detached Entity
__A *detached entity* is just an ordinary entity POJO__ whose identity value corresponds to a database row. The difference from a managed entity is that it’s __not tracked anymore by any *persistence context*__.

An entity can become detached when the _Session_ used to load it was closed, or when we call _Session.evict(entity)_ or _Session.clear()_.

Let’s see it in the code:
```java
FootballPlayer cr7 = session.get(FootballPlayer.class, 1L);

assertThat(getManagedEntities(session)).size().isEqualTo(1);
assertThat(getManagedEntities(session).get(0).getId()).isEqualTo(cr7.getId());

session.evict(cr7);

assertThat(getManagedEntities(session)).size().isEqualTo(0);
```

Our persistence context will not track the changes in detached entities:
```java
cr7.setName("CR7");
transaction.commit();

assertThat(getDirtyEntities()).isEmpty();
```

_Session.merge(entity)/Session.update(entity)_ can (re)attach a session:
```java
FootballPlayer messi = session.get(FootballPlayer.class, 2L);

session.evict(messi);
messi.setName("Leo Messi");
transaction.commit();

assertThat(getDirtyEntities()).isEmpty();

transaction = startTransaction(session);
session.update(messi);
transaction.commit();

assertThat(getDirtyEntities()).size().isEqualTo(1);
assertThat(getDirtyEntities().get(0).getName()).isEqualTo("Leo Messi");
```

For reference on both _Session.merge()_ and _Session.update()_ see [here](https://www.baeldung.com/hibernate-save-persist-update-merge-saveorupdate).

### 5.1. The Identity Field is All That Matters
Let’s have a look at the following logic:
```java
FootballPlayer gigi = new FootballPlayer();
gigi.setId(3);
gigi.setName("Gigi the Legend");
session.update(gigi);
```

In the example above, we’ve instantiated an entity in the usual way via its constructor. We’ve populated the fields with values and we’ve set the identity to 3, which corresponds to the identity of persistent data that belongs to Gigi Buffon. Calling _update()_ has exactly the same effect as if we’ve loaded the entity from another _persistence context_.

In fact, _Session_ doesn’t distinguish where a re-attached entity originated from.

It’s quite a common scenario in web applications to construct detached entities from HTML form values.

As far as _Session_ is concerned, a detached entity is just a plain entity whose identity value corresponds to persistent data.

Be aware that the example above just serves a demo purpose. and we need to know exactly what we’re doing. Otherwise, we could end up with null values across our entity if we just set the value on the field we want to update, leaving the rest untouched (so, effectively null).

## 6. Transient Entity
__A transient entity is simply an *entity* object that has no representation in the persistent store__ and is not managed by any _Session_.

A typical example of a transient entity would be instantiating a new entity via its constructor.

To make a transient entity _persistent_, we need to call _Session.save(entity)_ or _Session.saveOrUpdate(entity)_:
```java
FootballPlayer neymar = new FootballPlayer();
neymar.setName("Neymar");
session.save(neymar);

assertThat(getManagedEntities(session)).size().isEqualTo(1);
assertThat(neymar.getId()).isNotNull();

int count = queryCount("select count(*) from Football_Player where name='Neymar'");

assertThat(count).isEqualTo(0);

transaction.commit();
count = queryCount("select count(*) from Football_Player where name='Neymar'");

assertThat(count).isEqualTo(1);
```

As soon as we execute _Session.save(entity)_, the entity is assigned an identity value and becomes managed by the _Session_. However, it might not yet be available in the database as the INSERT operation might be delayed until the end of the unit of work.

## 7. Deleted Entity
__An entity is in a deleted (removed) state if *Session.delete(entity)*__ has been called, and the _Session_ has marked the entity for deletion. The DELETE command itself might be issued at the end of the unit of work.

Let’s see it in the following code:
```java
session.delete(neymar);

assertThat(getManagedEntities(session).get(0).getStatus()).isEqualTo(Status.DELETED);
```

However, notice that the entity stays in the persistent context store until the end of the unit of work.

## 8. Conclusion
The concept of _persistence context_ is central to understanding the lifecycle of Hibernate entities. We’ve clarified the lifecycle by looking into the code examples demonstrating each status.

As usual, the code used in this article can be found over on [GitHub](https://github.com/eugenp/tutorials/tree/master/persistence-modules/hibernate5).
