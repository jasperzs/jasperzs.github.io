---
layout:     post
title:      Spring Security - Authentication with a Database-backend UserDetailsService
subtitle:   Understanding Spring Security Context Propagation
date:       2019-01-18
author:     Mr.Humorous 🥘
header-img: img/springFramework/header.jpg
catalog: true
tags:
    - Spring Framework
    - Spring MVC
---

## 1. Overview
The example to create a custom database-backend _UserDetailsService_ for authentication with Spring Security is on [Github](https://github.com/eugenp/tutorials/tree/master/spring-security-mvc-boot).

## 2. UserDetailsService
The _UserDetailsService_ interface is used to retrieve user-related data. It has one method named _loadUserByUsername()_ which finds a user entity based on the username and can be overridden to customize the process of finding the user.

It is used by the _DaoAuthenticationProvider_ to load details about the user during authentication.

## 3. The User Model
For storing users, we will create a _User_ entity that is mapped to a database table, with the following attributes:
```java
@Entity
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    @Column(nullable = false, unique = true)
    private String username;

    private String password;

    //standard getters and setters
}
```

## 4. Retrieving a User
For the purpose of retrieving a user associated with a username, we will create a _DAO_ class using _Spring Data_ by extending the _JpaRepository_ interface:
```java
public interface UserRepository extends JpaRepository<User, Long> {

    User findByUsername(String username);
}
```

## 5. The UserDetailsService
In order to provide our own user service, we will need to implement the _UserDetailsService_ interface.

We’ll create a class called _MyUserDetailsService_ that overrides the method _loadUserByUsername()_ of the interface.

In this method, we retrieve the _User_ object using the DAO, and if it exists, wrap it into a _MyUserPrincipal_ object, which implements _UserDetails_, and returns it:
```java
@Service
public class MyUserDetailsService implements UserDetailsService {

    @Autowired
    private UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) {
        User user = userRepository.findByUsername(username);
        if (user == null) {
            throw new UsernameNotFoundException(username);
        }
        return new MyUserPrincipal(user);
    }
}
```

The _MyUserPrincipal_ class is defined as follows:
```java
public class MyUserPrincipal implements UserDetails {
    private User user;

    public MyUserPrincipal(User user) {
        this.user = user;
    }
    //...
}
```

## 6. Spring Configuration

## 6.1. Annotation Configuration
Using Spring annotations, we will define the _UserDetailsService_ bean, and set it as a property of the _authenticationProvider_ bean, which we inject into the _authenticationManager_:
```java
@Autowired
private MyUserDetailsService userDetailsService;

@Override
protected void configure(AuthenticationManagerBuilder auth)
  throws Exception {
    auth.authenticationProvider(authenticationProvider());
}

@Bean
public DaoAuthenticationProvider authenticationProvider() {
    DaoAuthenticationProvider authProvider
      = new DaoAuthenticationProvider();
    authProvider.setUserDetailsService(userDetailsService);
    authProvider.setPasswordEncoder(encoder());
    return authProvider;
}

@Bean
public PasswordEncoder encoder() {
    return new BCryptPasswordEncoder(11);
}
```

## 6.2. XML Configuration
For the XML configuration, we need to define a bean with type _MyUserDetailsService_, and inject it into Spring’s _authentication-provider_ bean:
```xml
<bean id="myUserDetailsService"
  class="org.baeldung.security.MyUserDetailsService"/>

<security:authentication-manager>
    <security:authentication-provider
      user-service-ref="myUserDetailsService" >
        <security:password-encoder ref="passwordEncoder">
        </security:password-encoder>
    </security:authentication-provider>
</security:authentication-manager>

<bean id="passwordEncoder"
  class="org.springframework.security
  .crypto.bcrypt.BCryptPasswordEncoder">
    <constructor-arg value="11"/>
</bean>
```
