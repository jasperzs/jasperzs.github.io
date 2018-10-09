---
layout:     post
title:      Guide to Spring Session
subtitle:   Spring Session Introduction
date:       2019-01-21
author:     Mr.Humorous 🥘
header-img: img/springFramework/header.jpg
catalog: true
tags:
    - Spring Framework
    - Spring MVC
    - Spring Boot
---

## 1. Overview
_Spring Session_ has the simple goal of free up session management from the limitations of the HTTP session stored in the server.

The solution makes it easy to share session data between services in the cloud without being tied to a single container (i.e. Tomcat). Additionally, it supports multiple sessions in the same browser and sending sessions in a header.

In this article, we’ll use _Spring Session_ to manage authentication information in a web app. While _Spring Session_ can persist data using JDBC, Gemfire, or MongoDB, we will use _Redis_.

For an introduction to _Redis_ check out [this](https://www.baeldung.com/spring-data-redis-tutorial) article.

## 2. A Simple Project
Let’s first create a simple _Spring Boot_ project to use as a base for our session examples later on:
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.4.0.RELEASE</version>
    <relativePath/>
</parent>

<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

Our application runs with _Spring Boot_ and the parent pom provides versions for each entry.

Let’s also add some configuration properties for our Redis server in application.properties:
```yaml
spring.redis.host=localhost
spring.redis.port=6379
```

## 3. Spring Boot Configuration
First, let’s demonstrate configuring _Spring Session_ with Boot.

### 3.1 Dependencies
Add these dependencies to our project:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session</artifactId>
</dependency>
```

We are using the boot parent pom to set the versions here, so these are guaranteed to work with our other dependencies.

### 3.2. Spring Session Configuration
Now let’s add a configuration class for _Spring Session_:
```java
@Configuration
@EnableRedisHttpSession
public class SessionConfig extends AbstractHttpSessionApplicationInitializer {
}
```

## 4. Standard Spring Config (no Boot)

### 4.1. Dependencies
First, if we’re adding _spring-session_ to a standard Spring project, we’ll need to explicitly define:
```xml
<dependency>
    <groupId>org.springframework.session</groupId>
    <artifactId>spring-session</artifactId>
    <version>1.2.2.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-redis</artifactId>
    <version>1.5.0.RELEASE</version>
</dependency>
```

### 4.2. Spring Session Configuration
Now let’s add a configuration class for _Spring Session_:
```java
@Configuration
@EnableRedisHttpSession
public class SessionConfig extends AbstractHttpSessionApplicationInitializer {
    @Bean
    public JedisConnectionFactory connectionFactory() {
        return new JedisConnectionFactory();
    }
}
```

As you can see the differences are minimal – we just have to now define our _JedisConnectionFactory_ bean explicitly – Boot does it for us.

In both types _@EnableRedisHttpSession_ and the extension of _AbstractHttpSessionApplicationInitializer_ will create and wire up a filter in front of all our security infrastructure to look for active sessions and populate the security context from values stored in _Redis_.

Let’s now complete this application with a controller and the security config.

## 5. Application Configuration
Navigate to our main application file and add a controller:
```java
@RestController
public class SessionController {
    @RequestMapping("/")
    public String helloAdmin() {
        return "hello admin";
    }
}
```

This will give us an endpoint to test.

Next, add our security configuration class:
```java
@Configuration
@EnableWebSecurity
public class SecurityConfig extends WebSecurityConfigurerAdapter {

    @Autowired
    public void configureGlobal(AuthenticationManagerBuilder auth) throws Exception {
        auth
          .inMemoryAuthentication()
          .withUser("admin").password("password").roles("ADMIN");
    }

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http
          .httpBasic().and()
          .authorizeRequests()
          .antMatchers("/").hasRole("ADMIN")
          .anyRequest().authenticated();
    }
}
```

This protects our endpoints with basic authentication and sets up a user to test with.

## 6. Test
Finally, let’s test everything out – we’ll define a simple test here that’s going to allow us to do 2 things:
- consume the live web application
- talk to Redis

Let’s first set things up:
```java
public class SessionControllerTest {

    private Jedis jedis;
    private TestRestTemplate testRestTemplate;
    private TestRestTemplate testRestTemplateWithAuth;
    private String testUrl = "http://localhost:8080/";

    @Before
    public void clearRedisData() {
        testRestTemplate = new TestRestTemplate();
        testRestTemplateWithAuth = new TestRestTemplate("admin", "password", null);

        jedis = new Jedis("localhost", 6379);
        jedis.flushAll();
    }
}
```

Notice how we’re setting up both of these clients – the HTTP client and the Redis one. Of course, at this point the server (and Redis) should be up and running – so that we can communicate with them via these tests.

Let’s begin by testing that _Redis_ is empty:
```java
@Test
public void testRedisIsEmpty() {
    Set<String> result = jedis.keys("*");
    assertEquals(0, result.size());
}
```

Now test that our security returns a 401 for unauthenticated requests:
```java
@Test
public void testUnauthenticatedCantAccess() {
    ResponseEntity<String> result = testRestTemplate.getForEntity(testUrl, String.class);
    assertEquals(HttpStatus.UNAUTHORIZED, result.getStatusCode());
}
```

Next, we test that _Spring Session_ is managing our authentication token:
```java
@Test
public void testRedisControlsSession() {
    ResponseEntity<String> result = testRestTemplateWithAuth.getForEntity(testUrl, String.class);
    assertEquals("hello admin", result.getBody()); //login worked

    Set<String> redisResult = jedis.keys("*");
    assertTrue(redisResult.size() > 0); //redis is populated with session data

    String sessionCookie = result.getHeaders().get("Set-Cookie").get(0).split(";")[0];
    HttpHeaders headers = new HttpHeaders();
    headers.add("Cookie", sessionCookie);
    HttpEntity<String> httpEntity = new HttpEntity<>(headers);

    result = testRestTemplate.exchange(testUrl, HttpMethod.GET, httpEntity, String.class);
    assertEquals("hello admin", result.getBody()); //access with session works worked

    jedis.flushAll(); //clear all keys in redis

    result = testRestTemplate.exchange(testUrl, HttpMethod.GET, httpEntity, String.class);
    assertEquals(HttpStatus.UNAUTHORIZED, result.getStatusCode());
    //access denied after sessions are removed in redis
}
```

First, our test confirms that our request was successful using the admin authentication credentials.

Then we extract the session value from the response headers and use it as our authentication in our second request. We validate that, and then clear all the data in _Redis_.

Finally, we make another request using the session cookie and confirm that we are logged out. This confirms that _Spring Session_ is managing our sessions.

## 7. Conclusion
_Spring Session_ is a powerful tool for managing HTTP sessions. With our session storage simplified to a configuration class and a few Maven dependencies, we can now wire up multiple applications to the same _Redis_ instance and share authentication information.

Full example code is on [Github](https://github.com/eugenp/tutorials/tree/master/spring-session).
