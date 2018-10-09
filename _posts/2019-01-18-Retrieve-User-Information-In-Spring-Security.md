---
layout:     post
title:      Spring Security - Retrieve User Information
subtitle:   Understanding how to retrieve user information
date:       2019-01-18
author:     Mr.Humorous 🥘
header-img: img/springFramework/header.jpg
catalog: true
tags:
    - Spring Framework
    - Spring MVC
---

## 1. Get the User in a Bean
The simplest way to __retrieve the currently authenticated principal__ is via a static call to the _SecurityContextHolder_:
```java
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
String currentPrincipalName = authentication.getName();
```

An improvement to this snippet is first __checking if there is an authenticate user__ before trying to access it:
```java
Authentication authentication = SecurityContextHolder.getContext().getAuthentication();
if (!(authentication instanceof AnonymousAuthenticationToken)) {
    String currentUserName = authentication.getName();
    return currentUserName;
}
```

There are of course downsides to having a static call like this – _decreased testability_ of the code being one of the more obvious. Instead, we will explore alternative solutions for this very common requirement.

## 2. Get the User in a Controller
In a _@Controller_ annotated bean, there are additional options – __the principal can be defined directly as a method argument__ and it will be correctly resolved by the framework:
```java
import java.security.Principal;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class SecurityController {

    @RequestMapping(value = "/username", method = RequestMethod.GET)
    @ResponseBody
    public String currentUserName(Principal principal) {
        return principal.getName();
    }
}
```

Alternatively, the authentication token can also be used:
```java
import org.springframework.security.core.Authentication;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class SecurityController {

    @RequestMapping(value = "/username", method = RequestMethod.GET)
    @ResponseBody
    public String currentUserName(Authentication authentication) {
        return authentication.getName();
    }
}
```

The API of the _Authentication_ class is very open so that the framework remains as flexible as possible. Because of this, the Spring Security principal can only be retrieved as an Object and needs to be cast to the correct __UserDetails instance__:
```java
UserDetails userDetails = (UserDetails) authentication.getPrincipal();
System.out.println("User has authorities: " + userDetails.getAuthorities());
```

And finally, directly __from the HTTP request__:
```java
import java.security.Principal;

import javax.servlet.http.HttpServletRequest;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
public class SecurityController {

    @RequestMapping(value = "/username", method = RequestMethod.GET)
    @ResponseBody
    public String currentUserNameSimple(HttpServletRequest request) {
        Principal principal = request.getUserPrincipal();
        return principal.getName();
    }
}
```

## 4. Get the User via a Custom Interface
To fully leverage the Spring dependency injection and be able to retrieve the authentication everywhere, not just in _@Controller_ beans, we need to hide the static access behind a simple facade:
```java
public interface IAuthenticationFacade {
    Authentication getAuthentication();
}
@Component
public class AuthenticationFacade implements IAuthenticationFacade {

    @Override
    public Authentication getAuthentication() {
        return SecurityContextHolder.getContext().getAuthentication();
    }
}
```

The facade exposes the Authentication object while hiding the static state and keeping the code decoupled and fully testable:
```java
@Controller
public class SecurityController {
    @Autowired
    private IAuthenticationFacade authenticationFacade;

    @RequestMapping(value = "/username", method = RequestMethod.GET)
    @ResponseBody
    public String currentUserNameSimple() {
        Authentication authentication = authenticationFacade.getAuthentication();
        return authentication.getName();
    }
}
```

## 5. Get the User in JSP
The currently authenticated principal __can also be access in JSP pages__, by leveraging the spring security taglib support. First, we need to define the tag in the page:
```xml
<%@ taglib prefix="security" uri="http://www.springframework.org/security/tags" %>
```

Next, we can __refer to the principal__:
```xml
<security:authorize access="isAuthenticated()">
    authenticated as <security:authentication property="principal.username" />
</security:authorize>
```
