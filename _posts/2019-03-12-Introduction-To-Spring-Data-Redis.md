---
layout:     post
title:      Introduction to Spring Data Redis
subtitle:   Use Spring Data with Redis
date:       2019-03-12
author:     Mr.Humorous 🥘
header-img: img/springFramework/header.jpg
catalog: true
tags:
    - Spring Framework
    - Spring Data
---

## 1. Overview
This article is __an introduction to Spring Data Redis__, which provides the abstractions of the Spring Data platform to [Redis](http://redis.io/) – the popular in-memory data structure store.

Redis is driven by a keystore-based data structure to persist data and can be used as a database, cache, message broker, etc.

We’ll be able to use the common patterns of Spring Data (templates, etc.), while also having the traditional simplicity of all Spring Data projects.

## 2. Maven Dependencies
Let’s start by declaring the Spring Data Redis dependencies in the _pom.xml_:
```xml
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>2.0.3.RELEASE</version>
 </dependency>

<dependency>
    <groupId>redis.clients</groupId>
    <artifactId>jedis</artifactId>
    <version>2.9.0</version>
    <type>jar</type>
</dependency>
```

## 3. The Redis Configuration
To define the connection settings between the application client and the Redis server instance, we need to use a Redis client.

There is a number of Redis client implementations available for Java. In this tutorial, __we’ll use [Jedis](https://github.com/xetorthio/jedis) – a simple and powerful Redis client implementation.__

There is good support for both XML and Java configuration in the framework; for this tutorial, we’ll use Java-based configuration.

### 3.1. Java Configuration
Let’s start with the configuration bean definitions:
```java
@Bean
JedisConnectionFactory jedisConnectionFactory() {
    return new JedisConnectionFactory();
}

@Bean
public RedisTemplate<String, Object> redisTemplate() {
    RedisTemplate<String, Object> template = new RedisTemplate<>();
    template.setConnectionFactory(jedisConnectionFactory());
    return template;
}
```

The configuration is quite simple. First, using the Jedis client, we’re defining a _connectionFactory_.

Then, we defined a _RedisTemplate_ using the _jedisConnectionFactory_. This can be used for querying data with a custom repository.

### 3.2. Custom Connection Properties
You may have already noticed that the usual connection-related properties are missing in the above configuration. For example, the server address and port are missing in the configuration.The reason is simple: for our example, we’re using the defaults.

However, if we need to configure the connection details, we can always modify the _jedisConnectionFactory_ configuration as follows:
```java
@Bean
JedisConnectionFactory jedisConnectionFactory() {
    JedisConnectionFactory jedisConFactory
      = new JedisConnectionFactory();
    jedisConFactory.setHostName("localhost");
    jedisConFactory.setPort(6379);
    return jedisConFactory;
}
```

## 4. Redis Repository
Let’s use a Student entity for our examples:
```java
@RedisHash("Student")
public class Student implements Serializable {

    public enum Gender {
        MALE, FEMALE
    }

    private String id;
    private String name;
    private Gender gender;
    private int grade;
    // ...
}
```

### 4.1. The Spring Data Repository
Let’s now create the StudentRepository as follows:
```java
@Repository
public interface StudentRepository extends CrudRepository<Student, String> {}
```

## 5. Data Access using StudentRepository
__By extending *CrudRepository* in *StudentRepository*, we automatically get a complete set of persistence methods that perform CRUD functionality.__

### 5.1. Saving a New Student Object
Let’s save a new student object in the data store:
```java
Student student = new Student(
  "Eng2015001", "John Doe", Student.Gender.MALE, 1);
studentRepository.save(student);
```

### 5.2. Retrieving an Existing Student Object
We can verify the correct insertion of the student in the previous section by fetching the student data:
```java
Student retrievedStudent =
  studentRepository.findById("Eng2015001").get();
```

### 5.3. Updating an Existing Student Object
Let’s change the name of the student retrieved above and save it again:
```java
retrievedStudent.setName("Richard Watson");
studentRepository.save(student);
```

Finally, we can retrieve the student’s data again and verify that the name is updated in the datastore.

### 5.4. Deleting an Existing Student Data
We can delete the above-inserted student data:
```java
studentRepository.deleteById(student.getId());
```

Now we can search for the student object and verify that the result is _null_.

### 5.5. Find All Student Data
We can insert a few student objects:
```java
Student engStudent = new Student(
  "Eng2015001", "John Doe", Student.Gender.MALE, 1);
Student medStudent = new Student(
  "Med2015001", "Gareth Houston", Student.Gender.MALE, 2);
studentRepository.save(engStudent);
studentRepository.save(medStudent);
```

We can also achieve this by inserting a collection. For that, there is a different method – _saveAll()_ – which accepts a single _Iterable_ object containing multiple student objects that we want to persist.

To find all inserted students, we can use the _findAll()_ method:
```java
List<Student> students = new ArrayList<>();
studentRepository.findAll().forEach(students::add);
```

Then we can quickly check the size of the students _list_ or verify for a greater granularity by checking the properties of each object.

## 6. Conclusion
In this tutorial, we went through the basics of Spring Data Redis. The source code of the examples above can be found in a [GitHub project](https://github.com/eugenp/tutorials/tree/master/persistence-modules/spring-data-redis).
