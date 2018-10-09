---
layout:     post
title:      Introduction to Spring Method Security
subtitle:   Understanding Spring Security
date:       2019-01-16
author:     Mr.Humorous 🥘
header-img: img/springFramework/header.jpg
catalog: true
tags:
    - Spring Framework
    - Spring MVC
    - Spring Boot
---

## 1. Introduction
Simply put, Spring Security supports authorization semantics at the method level.

Typically, we could secure our service layer by, for example, restricting which roles are able to execute a particular method – and test it using dedicated method-level security test support.

In this article, we’re going to review the use of some security annotations first. Then, we’ll focus on testing our method security with different strategies.

## 2. Enabling Method Security
First of all, to use Spring Method Security, we need to add the spring-security-config dependency:
```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-config</artifactId>
</dependency>
```

If we want to use Spring Boot, we can use the _spring-boot-starter-security_ dependency which includes _spring-security-config_:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

__Next, we need to enable global Method Security:__
```java
@Configuration
@EnableGlobalMethodSecurity(
  prePostEnabled = true,
  securedEnabled = true,
  jsr250Enabled = true)
public class MethodSecurityConfig
  extends GlobalMethodSecurityConfiguration {
}
```
- The _prePostEnabled_ property enables Spring Security pre/post annotations
- The _securedEnabled_ property determines if the @Secured annotation should be enabled
- The _jsr250Enabled_ property allows us to use the @RoleAllowed annotation

## 3. Applying Method Security

### 3.1 Using @Secured Annotation
__The @Secured annotation is used to specify a list of roles on a method__. Hence, a user only can access that method if she has at least one of the specified roles.

Let’s define a _getUsername_ method:
```java
@Secured("ROLE_VIEWER")
public String getUsername() {
    SecurityContext securityContext = SecurityContextHolder.getContext();
    return securityContext.getAuthentication().getName();
}
```

Here, the _@Secured(“ROLE_VIEWER”)_ annotation defines that only users who have the role _ROLE_VIEWER_ are able to execute the _getUsername_ method.

Besides, we can define a list of roles in a _@Secured_ annotation:
```java
@Secured({ "ROLE_VIEWER", "ROLE_EDITOR" })
public boolean isValidUsername(String username) {
    return userRoleRepository.isValidUsername(username);
}
```

In this case, the configuration states that if a user has either _ROLE_VIEWER_ or _ROLE_EDITOR_, that user can invoke the _isValidUsername_ method.

__The @Secured annotation doesn’t support Spring Expression Language (SpEL).__

### 3.2 Using @RoleAllowed Annotation
__The @RoleAllowed annotation is the JSR-250’s equivalent annotation of the @Secured annotation.__

Basically, we can use the _@RoleAllowed_ annotation in a similar way as _@Secured_. Thus, we could re-define getUsername and _isValidUsername_ methods:
```java
@RolesAllowed("ROLE_VIEWER")
public String getUsername2() {
    //...
}

@RolesAllowed({ "ROLE_VIEWER", "ROLE_EDITOR" })
public boolean isValidUsername2(String username) {
    //...
}
```

Similarly, only the user who has role _ROLE_VIEWER_ can execute _getUsername2_.

Again, a user is able to invoke _isValidUsername2_ only if she has at least one of _ROLE_VIEWER_ or _ROLER_EDITOR_ roles.

### 3.3. Using @PreAuthorize and @PostAuthorize Annotations
__Both @PreAuthorize and @PostAuthorize annotations provide expression-based access control.__ Hence, predicates can be written using [SpEL (Spring Expression Language)](https://www.baeldung.com/spring-expression-language).

__The @PreAuthorize annotation checks the given expression before entering the method,__ whereas, __the @PostAuthorize annotation verifies it after the execution of the method and could alter the result__.

Now, let’s declare a _getUsernameInUpperCase_ method as below:
```java
@PreAuthorize("hasRole('ROLE_VIEWER')")
public String getUsernameInUpperCase() {
    return getUsername().toUpperCase();
}
```

The _@PreAuthorize(“hasRole(‘ROLE_VIEWER’)”)_ has the same meaning as _@Secured(“ROLE_VIEWER”)_ which we used in the previous section. Feel free to discover more security expressions details in previous articles. Feel free to discover more [security expressions details in previous articles](https://www.baeldung.com/spring-security-expressions-basic).

Consequently, the annotation _@Secured({“ROLE_VIEWER”,”ROLE_EDITOR”})_ can be replaced with _@PreAuthorize(“hasRole(‘ROLE_VIEWER’) or hasRole(‘ROLE_EDITOR’)”)_:
```java
@PreAuthorize("hasRole('ROLE_VIEWER') or hasRole('ROLE_EDITOR')")
public boolean isValidUsername3(String username) {
    //...
}
```

Moreover, __we can actually use the method argument as part of the expression__:
```java
@PreAuthorize("#username == authentication.principal.username")
public String getMyRoles(String username) {
    //...
}
```

Here, a user can invoke the _getMyRoles_ method only if the value of the argument _username_ is the same as current principal’s username.

__It’s worth to note that @PreAuthorize expressions can be replaced by @PostAuthorize ones__.

Let’s rewrite _getMyRoles_:
```java
@PostAuthorize("#username == authentication.principal.username")
public String getMyRoles2(String username) {
    //...
}
```

In the previous example, however, the authorization would get delayed after the execution of the target method.

Additionally, __the @PostAuthorize annotation provides the ability to access the method result__:
```java
@PostAuthorize
  ("returnObject.username == authentication.principal.nickName")
public CustomUser loadUserDetail(String username) {
    return userRoleRepository.loadUserByUserName(username);
}
```

In this example, the _loadUserDetail_ method would only execute successfully if the username of the returned _CustomUser_ is equal to the current authentication principal’s nickname.

In this section, we mostly use simple Spring expressions. For more complex scenarios, we could create [custom security expressions](https://www.baeldung.com/spring-security-create-new-custom-security-expression).

### 3.4. Using @PreFilter and @PostFilter Annotations
__Spring Security provides the @PreFilter annotation to filter a collection argument before executing the method__:
```java
@PreFilter("filterObject != authentication.principal.username")
public String joinUsernames(List<String> usernames) {
    return usernames.stream().collect(Collectors.joining(";"));
}
```

In this example, we’re joining all usernames except for the one who is authenticated.

Here, __our expression uses the name filterObject to represent the current object in the collection__.

However, if the method has more than one argument which is a collection type, we need to use the _filterTarget_ property to specify which argument we want to filter:
```java
@PreFilter
  (value = "filterObject != authentication.principal.username",
  filterTarget = "usernames")
public String joinUsernamesAndRoles(
  List<String> usernames, List<String> roles) {

    return usernames.stream().collect(Collectors.joining(";"))
      + ":" + roles.stream().collect(Collectors.joining(";"));
}
```

Additionally, __we can also filter the returned collection of a method by using @PostFilter annotation__:
```java
@PostFilter("filterObject != authentication.principal.username")
public List<String> getAllUsernamesExceptCurrent() {
    return userRoleRepository.getAllUsernames();
}
```

In this case, the name _filterObject_ refers to the current object in the returned collection.

With that configuration, Spring Security will iterate through the returned list and remove any value which matches with the principal’s username.

More detail of _@PreFilter_ and _@PostFilter_ can be found in the [Spring Security – @PreFilter and @PostFilter](https://www.baeldung.com/spring-security-prefilter-postfilter) article.

### 3.5. Method Security Meta-Annotation
We typically find ourselves in a situation where we protect different methods using the same security configuration.

In this case, we can define a security meta-annotation:
```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@PreAuthorize("hasRole('VIEWER')")
public @interface IsViewer {
}
```

Next, we can directly use the @IsViewer annotation to secure our method:
```java
@IsViewer
public String getUsername4() {
    //...
}
```

Security meta-annotations are a great idea because they add more semantics and decouple our business logic from the security framework.

### 3.6. Security Annotation at the Class Level
If we find ourselves using the same security annotation for every method within one class, we can consider putting that annotation at class level:
```java
@Service
@PreAuthorize("hasRole('ROLE_ADMIN')")
public class SystemService {

    public String getSystemYear(){
        //...
    }

    public String getSystemDate(){
        //...
    }
}
```

In above example, the security rule _hasRole(‘ROLE_ADMIN’)_ will be applied to both _getSystemYear_ and _getSystemDate_ methods.

### 3.7. Multiple Security Annotations on a Method
We can also use multiple security annotations on one method:
```java
@PreAuthorize("#username == authentication.principal.username")
@PostAuthorize("returnObject.username == authentication.principal.nickName")
public CustomUser securedLoadUserDetail(String username) {
    return userRoleRepository.loadUserByUserName(username);
}
```

Hence, Spring will verify authorization both before and after the execution of the _securedLoadUserDetail_ method.

## 4. Important Considerations
There are two points we’d like to remind regarding method security:
- __By default, Spring AOP proxying is used to apply method security__ – if a secured method A is called by another method within the same class, security in A is ignored altogether. This means method A will execute without any security checking. The same applies to private methods
- __Spring *SecurityContext* is thread-bound__ – by default, the security context isn’t propagated to child-threads. For more information, we can refer to [Spring Security Context Propagation](https://www.baeldung.com/spring-security-async-principal-propagation) article

## 5. Testing Method Security

### 5.1. Configuration
__To test Spring Security with JUnit, we need the spring-security-test dependency:__
```xml
<dependency>
    <groupId>org.springframework.security</groupId>
    <artifactId>spring-security-test</artifactId>
</dependency>
```

We don’t need to specify the dependency version because we’re using the Spring Boot plugin. Latest versions of this dependency can be found on Maven Central.

Next, let’s configure a simple Spring Integration test by specifying the runner and the ApplicationContext configuration:
```java
@RunWith(SpringRunner.class)
@ContextConfiguration
public class TestMethodSecurity {
    // ...
}
```

### 5.2. Testing Username and Roles
Now that our configuration is ready, let’s try to test our _getUsername_ method which is secured by the annotation _@Secured(“ROLE_VIEWER”)_:
```java
@Secured("ROLE_VIEWER")
public String getUsername() {
    SecurityContext securityContext = SecurityContextHolder.getContext();
    return securityContext.getAuthentication().getName();
}
```

Since we use the _@Secured_ annotation here, it requires a user to be authenticated to invoke the method. Otherwise, we’ll get an _AuthenticationCredentialsNotFoundException_.

Hence, __we need to provide a user to test our secured method. To achieve this, we decorate the test method with *@WithMockUser* and provide a user and roles__:
```java
@Test
@WithMockUser(username = "john", roles = { "VIEWER" })
public void givenRoleViewer_whenCallGetUsername_thenReturnUsername() {
    String userName = userRoleService.getUsername();

    assertEquals("john", userName);
}
```

We’ve provided an authenticated user whose username is _john_ and whose role is _ROLE_VIEWER_. If we don’t specify the _username_ or _role_, the default _username_ is _user_ and default _role_ is _ROLE_USER_.

__Note that it isn’t necessary to add the ROLE_ prefix here, Spring Security will add that prefix automatically.__

If we don’t want to have that prefix, we can consider using _authority_ instead of _role_.

For example, let’s declare a _getUsernameInLowerCase_ method:
```java
@PreAuthorize("hasAuthority('SYS_ADMIN')")
public String getUsernameLC(){
    return getUsername().toLowerCase();
}
```

We could test that using authorities:
```java
@Test
@WithMockUser(username = "JOHN", authorities = { "SYS_ADMIN" })
public void givenAuthoritySysAdmin_whenCallGetUsernameLC_thenReturnUsername() {
    String username = userRoleService.getUsernameInLowerCase();

    assertEquals("john", username);
}
```

Conveniently, __if we want to use the same user for many test cases, we can declare the *@WithMockUser* annotation at test class__:
```java
@RunWith(SpringRunner.class)
@ContextConfiguration
@WithMockUser(username = "john", roles = { "VIEWER" })
public class TestWithMockUserAtClassLevel {
    //...
}
```

__If we wanted to run our test as an anonymous user, we could use the @WithAnonymousUser annotation:__
```java
@Test(expected = AccessDeniedException.class)
@WithAnonymousUser
public void givenAnomynousUser_whenCallGetUsername_thenAccessDenied() {
    userRoleService.getUsername();
}
```

In the example above, we expect an _AccessDeniedException_ because the anonymous user isn’t granted the role _ROLE_VIEWER_ or the authority _SYS_ADMIN_.

### 5.3. Testing with a Custom UserDetailsService

__For most applications, it’s common to use a custom class as authentication principal__. In this case, the custom class needs to implement the _org.springframework.security.core.userdetails.UserDetails_ interface.

In this article, we declare a _CustomUser_ class which extends the existing implementation of _UserDetails_, which is _org.springframework.security.core.userdetails.User_:
```java
public class CustomUser extends User {
    private String nickName;
    // getter and setter
}
```

Let’s take back the example with the @PostAuthorize annotation in section 3:
```java
@PostAuthorize("returnObject.username == authentication.principal.nickName")
public CustomUser loadUserDetail(String username) {
    return userRoleRepository.loadUserByUserName(username);
}
```

In this case, the method would only execute successfully if the _username_ of the returned _CustomUser_ is equal to the current authentication principal’s _nickname_.

If we wanted to test that method, __we could provide an implementation of UserDetailsService which could load our CustomUser based on the username__:
```java
@Test
@WithUserDetails(
  value = "john",
  userDetailsServiceBeanName = "userDetailService")
public void whenJohn_callLoadUserDetail_thenOK() {

    CustomUser user = userService.loadUserDetail("jane");

    assertEquals("jane", user.getNickName());
}
```

Here, the _@WithUserDetails_ annotation states that we’ll use a _UserDetailsService_ to initialize our authenticated user. The service is referred by the _userDetailsServiceBeanName_ property. This _UserDetailsService_ might be a real implementation or a fake for testing purposes.

Additionally, the service will use the value of the property _value_ as the username to load _UserDetails_.

Conveniently, we can also decorate with a _@WithUserDetails_ annotation at the class level, similarly to what we did with the _@WithMockUser_ annotation.

### 5.4. Testing with Meta Annotations
We often find ourselves reusing the same user/roles over and over again in various tests.

For these situations, it’s convenient to create a _meta-annotation_.

Taking back the previous example _@WithMockUser(username=”john”, roles={“VIEWER”})_, we can declare a meta-annotation as:
```java
@Retention(RetentionPolicy.RUNTIME)
@WithMockUser(value = "john", roles = "VIEWER")
public @interface WithMockJohnViewer { }
```

Then we can simply use _@WithMockJohnViewer_ in our test:
```java
@Test
@WithMockJohnViewer
public void givenMockedJohnViewer_whenCallGetUsername_thenReturnUsername() {
    String userName = userRoleService.getUsername();

    assertEquals("john", userName);
}
```

Likewise, we can use meta-annotations to create domain-specific users using _@WithUserDetails_.
