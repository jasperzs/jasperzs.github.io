---
layout:     post
title:      Spring Boot with Multiple SQL Import Files
subtitle:   Initialize database upon application startup
date:       2019-03-30
author:     Mr.Humorous 🥘
header-img: img/springFramework/header.jpg
catalog: true
tags:
    - Spring Boot
    - Spring Framework
---

## 1. Overview
Spring Boot allows us to import sample data into our database – mainly to prepare data for integration tests. Out of the box, there are two possibilities. __We can use *import.sql* (Hibernate support) or *data.sql* (Spring JDBC support) files to load data__.

However, sometimes we want to split one big SQL file into a few smaller ones, e.g., for better readability or to share some files with an init data between modules.

In this tutorial, we’ll show how to do it with both – Hibernate and Spring JDBC.

## 2. Hibernate Support
We can define files which contain __sample data to load with a property *spring.jpa.properties.hibernate.hbm2ddl.import_files*__. It can be set in the application.properties file inside the test resources folder.

This is in a case we want to load sample data just for JUnit tests. The value has to be a comma-separated list of files to import:
```
spring.jpa.properties.hibernate.hbm2ddl.import_files=import_active_users.sql,import_inactive_users.sql
```

This configuration will load sample data from two files: _import_active_users.sql_ and _import_inactive_users.sql_. Important to mention here is that we __have to use prefix *spring.jpa.properties* to pass values (JPA configuration) to the EntityManagerFactory__.

Next, we’ll show how we can do it with a Spring JDBC support.

## 3. Spring JDBC Support
The configuration for initial data and __Spring JDBC support is very similar to Hibernate. We have to use the *spring.datasource.data*__ property:
```
spring.datasource.data=import_active_users.sql,import_inactive_users.sql
```

Setting the value as above gives the same results as in the Hibernate support. However, a significant __advantage of this solution is a possibility to define value using an Ant-style pattern:__
```
spring.datasource.data=import_*_users.sql
```

The above value tells the Spring to search for all files with a name that matches *import_\*_users.sql* pattern and import data which is inside.

## 4. Conclusion
In this short article, we showed how to configure Spring Boot application to load initial data from custom SQL files.

Finally, we showed two possibilities – Hibernate and Spring JDBC. They both work pretty well, and it’s up to the developer which one to chose.

As always, the complete code examples used in this article is available over on [Github](https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-boot-persistence).
