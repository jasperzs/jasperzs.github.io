---
layout:     post
title:      Understanding Spring Data REST
subtitle:   Introduction to Spring Data REST
date:       2019-01-19
author:     Mr.Humorous 🥘
header-img: img/springFramework/header.jpg
catalog: true
tags:
    - Spring Framework
---

## 1. Overview
In general, Spring Data REST is built on top of the Spring Data project and makes it easy to build hypermedia-driven REST web services that connect to Spring Data repositories – all using HAL(HyperText Application Language) as the driving hypermedia type.

It takes away a lot of the manual work usually associated with such tasks and makes implementing basic CRUD functionality for web applications quite simple.

## 2. Maven Dependencies
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId
    <artifactId>spring-boot-starter-data-rest</artifactId></dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
```

## 3. Writing the Application
A domain object to represent a user of our website:
```java
@Entity
public class WebsiteUser {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private long id;

    private String name;
    private String email;

    // standard getters and setters
}
```

Every user has a name and an email, as well as an automatically-generated id. Now we can write a simple repository:
```java
@RepositoryRestResource(collectionResourceRel = "users", path = "users")
public interface UserRepository extends PagingAndSortingRepository<WebsiteUser, Long> {
    List<WebsiteUser> findByName(@Param("name") String name);
}
```

This is an interface that allows you to perform various operations with __WebsiteUser__ objects. We also defined a custom query that will provide a list of users based on a given name.

The __@RepositoryRestResource__ annotation is optional and is used to customize the REST endpoint. If we decided to omit it, Spring would automatically create an endpoint at __“/websiteUsers”__ instead of __“/users“__.

Finally, we will write a standard __Spring Boot main class to initialize the application__:
```java
@SpringBootApplication
public class SpringDataRestApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringDataRestApplication.class, args);
    }
}
```

## 4. Accessing the REST API
If we run the application and go to http://localhost:8080/ in a browser, we will receive the following JSON:
```
{
  "_links" : {
    "users" : {
      "href" : "http://localhost:8080/users{?page,size,sort}",
      "templated" : true
    },
    "profile" : {
      "href" : "http://localhost:8080/profile"
    }
  }
}
```
As you can see, there is a __“/users”__ endpoint available, and it already has the __“?page“__, __“?size”__ and __“?sort”__ options.

There is also a standard __“/profile”__ endpoint, which provides application metadata. It is important to note that the response is structured in a way that follows the constraints of the REST architecture style. Specifically, it provides a uniform interface and self-descriptive messages. This means that each message contains enough information to describe how to process the message.

There are no users in our application yet, so going to http://localhost:8080/users would just show an empty list of users. Let’s use curl to add a user.
```
$ curl -i -X POST -H "Content-Type:application/json" -d '{  "name" : "Test", \ 
"email" : "test@test.com" }' http://localhost:8080/users
{
  "name" : "test",
  "email" : "test@test.com",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/users/1"
    },
    "websiteUser" : {
      "href" : "http://localhost:8080/users/1"
    }
  }
}
```

Lets take a look at the response headers as well:
```
HTTP/1.1 201 Created
Server: Apache-Coyote/1.1
Location: http://localhost:8080/users/1
Content-Type: application/hal+json;charset=UTF-8
Transfer-Encoding: chunked
```

You will notice that the returned content type is _“application/hal+json“_. [HAL](http://stateless.co/hal_specification.html) is a simple format that gives a consistent and easy way to hyperlink between resources in your API. The header also automatically contains the _Location_ header, which is the address we can use to access the newly created user.

We can now access this user at http://localhost:8080/users/1
```
{
  "name" : "test",
  "email" : "test@test.com",
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/users/1"
    },
    "websiteUser" : {
      "href" : "http://localhost:8080/users/1"
    }
  }
}
```

You can also use curl or any other REST client to issue PUT, PATCH, and DELETE requests. It also is important to note that Spring Data REST automatically follows the principles of HATEOAS. [HATEOAS](https://spring.io/understanding/HATEOAS) is one of the constraints of the REST architecture style, and it means that hypertext should be used to find your way through the API.

Finally, lets try to access the custom query that we wrote earlier and find all users with the name “test”. This is done by going to http://localhost:8080/users/search/findByName?name=test
```
{
  "_embedded" : {
    "users" : [ {
      "name" : "test",
      "email" : "test@test.com",
      "_links" : {
        "self" : {
          "href" : "http://localhost:8080/users/1"
        },
        "websiteUser" : {
          "href" : "http://localhost:8080/users/1"
        }
      }
    } ]
  },
  "_links" : {
    "self" : {
      "href" : "http://localhost:8080/users/search/findByName?name=test"
    }
  }
}
```
