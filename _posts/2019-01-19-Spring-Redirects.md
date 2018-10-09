---
layout:     post
title:      Understanding Spring Redirects
subtitle:   Introduction to Spring Redirects
date:       2019-01-19
author:     Mr.Humorous 🥘
header-img: img/springFramework/header.jpg
catalog: true
tags:
    - Spring Framework
    - Spring MVC
---

## 1. Why Do A Redirect?
Let’s first consider __the reasons why you may need to do a redirect__ in a Spring application.

There are many possible examples and reasons of course. One simple one might be POSTing form data, working around the double submission problem, or just delegating the execution flow to another controller method.

A quick side note here is that the typical Post/Redirect/Get pattern doesn’t adequately address double submission issues – problems such as refreshing the page before the initial submission has completed may still result in a double submission.

## 2. Redirect with the RedirectView
```java
@Controller
@RequestMapping("/")
public class RedirectController {

    @GetMapping("/redirectWithRedirectView")
    public RedirectView redirectWithUsingRedirectView(
      RedirectAttributes attributes) {
        attributes.addFlashAttribute("flashAttribute", "redirectWithRedirectView");
        attributes.addAttribute("attribute", "redirectWithRedirectView");
        return new RedirectView("redirectedUrl");
    }
}
```

Behind the scenes, _RedirectView_ will trigger a _HttpServletResponse.sendRedirect()_ – which will perform the actual redirect.

Notice here how __we’re injecting the redirect attributes into the method__ – the framework will do the heavy lifting here and allow us to interact with these attributes.

We’re adding the model attribute _attribute_ – which will be exposed as HTTP query parameter. The model must contain only objects – generally Strings or objects that can be converted to Strings.

__Let’s now test our redirect – with the help of a simple curl command:__
```
curl -i http://localhost:8080/spring-rest/redirectWithRedirectView
```

The result will be:
```
HTTP/1.1 302 Found
Server: Apache-Coyote/1.1
Location:
  http://localhost:8080/spring-rest/redirectedUrl?attribute=redirectWithRedirectView
```

## 3. Redirect With the Prefix redirect:
The previous approach – using _RedirectView_ – is suboptimal for a few reasons.

First- we’re now coupled to the Spring API because we’re using the _RedirectView_ directly in our code.

Second – we now need to know from the start, when implementing that controller operation – that the result will always be a redirect – which may not always be the case.

A better option is using the prefix _redirect_: – the redirect view name is injected into the controller like any other logical view name. __The controller is not even aware that redirection is happening.__
```java
@Controller
@RequestMapping("/")
public class RedirectController {

    @GetMapping("/redirectWithRedirectPrefix")
    public ModelAndView redirectWithUsingRedirectPrefix(ModelMap model) {
        model.addAttribute("attribute", "redirectWithRedirectPrefix");
        return new ModelAndView("redirect:/redirectedUrl", model);
    }
}
```

When a view name is returned with the prefix _redirect_: – the _UrlBasedViewResolver_ (and all its subclasses) will recognize this as a special indication that a redirect needs to happen. The rest of the view name will be used as the redirect URL.

A quick but important note here is that – when we use this logical view name here – _redirect:/redirectedUrl_ – we’re doing a redirect __relative to the current Servlet context__.

We can use a name such as _a redirect: http://localhost:8080/spring-redirect-and-forward/redirectedUrl_ if we need to redirect to an absolute URL.

__So now, when we execute the curl command:__
```
curl -i http://localhost:8080/spring-rest/redirectWithRedirectPrefix
```

We’ll immediately get redirected:
```
HTTP/1.1 302 Found
Server: Apache-Coyote/1.1
Location:
  http://localhost:8080/spring-rest/redirectedUrl?attribute=redirectWithRedirectPrefix
```

## 4. Forward With the Prefix forward:
Before the code, let’s go over __a quick, high-level overview of the semantics of forward vs. redirect__:
- _redirect_ will respond with a 302 and the new URL in the _Location_ header; the browser/client will then make another request to the new URL
- _forward_ happens entirely on a server side; the Servlet container forwards the same request to the target URL; the URL won’t change in the browser

```java
@Controller
@RequestMapping("/")
public class RedirectController {

    @GetMapping("/forwardWithForwardPrefix")
    public ModelAndView redirectWithUsingForwardPrefix(ModelMap model) {
        model.addAttribute("attribute", "forwardWithForwardPrefix");
        return new ModelAndView("forward:/redirectedUrl", model);
    }
}
```

Same as _redirect_:, the _forward_: prefix will be resolved by _UrlBasedViewResolver_ and its subclasses. Internally, this will create an _InternalResourceView_ which does a _RequestDispatcher.forward()_ to the new view.

__When we execute the command with curl__:
```
curl -I http://localhost:8080/spring-rest/forwardWithForwardPrefix
```

We will get HTTP 405 (Method not allowed):
```
HTTP/1.1 405 Method Not Allowed
Server: Apache-Coyote/1.1
Allow: GET
Content-Type: text/html;charset=utf-8
```

To wrap up, compared to the two requests that we had in the case of the redirect solution, in this case, we only have a single request going out from the browser/client to the server side. The attribute that was previously added by the redirect is, of course, missing as well.

## 5. Attributes With the RedirectAttributes
Next – let’s look closer at __passing attributes in a redirect__ – making full use the framework with _RedirectAttribures_:
```java
@GetMapping("/redirectWithRedirectAttributes")
public RedirectView redirectWithRedirectAttributes(RedirectAttributes attributes) {

    attributes.addFlashAttribute("flashAttribute", "redirectWithRedirectAttributes");
    attributes.addAttribute("attribute", "redirectWithRedirectAttributes");
    return new RedirectView("redirectedUrl");
}
```

Notice also that we are __adding a flash attribute__ as well – this is an attribute that won’t make it into the URL. What we can achieve with this kind of attribute is – we can later access the flash attribute using _@ModelAttribute(“flashAttribute”)_ __only in the method that is the final target of the redirect__:
```java
@GetMapping("/redirectedUrl")
public ModelAndView redirection(
  ModelMap model,
  @ModelAttribute("flashAttribute") Object flashAttribute) {

     model.addAttribute("redirectionAttribute", flashAttribute);
     return new ModelAndView("redirection", model);
}
```

__So, to wrap up – if we test the functionality with curl:__
```
curl -i http://localhost:8080/spring-rest/redirectWithRedirectAttributes
```

We will be redirected to the new location:
```
HTTP/1.1 302 Found
Server: Apache-Coyote/1.1
Set-Cookie: JSESSIONID=4B70D8FADA2FD6C22E73312C2B57E381; Path=/spring-rest/; HttpOnly
Location: http://localhost:8080/spring-rest/redirectedUrl;
  jsessionid=4B70D8FADA2FD6C22E73312C2B57E381?attribute=redirectWithRedirectAttributes
```

That way, using _RedirectAttribures_ instead of a _ModelMap_ gives us the ability only to share __some attributes between the two methods__ that are involved in the redirect operation.

## 6. An Alternative Configuration Without the Prefix
Let’s now explore an alternative configuration – a redirect without using the prefix.

To achieve this, we need to use an _org.springframework.web.servlet.view.XmlViewResolver_:
```xml
<bean class="org.springframework.web.servlet.view.XmlViewResolver">
    <property name="location">
        <value>/WEB-INF/spring-views.xml</value>
    </property>
    <property name="order" value="0" />
</bean>
```

Instead of _org.springframework.web.servlet.view.InternalResourceViewResolver_ we used in the previous configuration:
```xml
<bean
  class="org.springframework.web.servlet.view.InternalResourceViewResolver">
</bean>
```

We also need to define a _RedirectView_ bean in the configuration:
```xml
<bean id="RedirectedUrl" class="org.springframework.web.servlet.view.RedirectView">
    <property name="url" value="redirectedUrl" />
</bean>
```

Now we can __trigger the redirect by referencing this new bean by id__:
```java
@Controller
@RequestMapping("/")
public class RedirectController {

    @GetMapping("/redirectWithXMLConfig")
    public ModelAndView redirectWithUsingXMLConfig(ModelMap model) {
        model.addAttribute("attribute", "redirectWithXMLConfig");
        return new ModelAndView("RedirectedUrl", model);
    }
}
```

__And to test it, we’ll again use the curl command:__
```
curl -i http://localhost:8080/spring-rest/redirectWithRedirectView
```

The result will be:
```
HTTP/1.1 302 Found
Server: Apache-Coyote/1.1
Location:
  http://localhost:8080/spring-rest/redirectedUrl?attribute=redirectWithRedirectView
```

## 7. Redirecting an HTTP POST Request
For use cases like bank payments, we might need to redirect an HTTP POST request. Depending upon the HTTP status code returned, POST request can be redirected to an HTTP GET or POST.

As per HTTP 1.1 protocol [reference](https://tools.ietf.org/html/rfc7238), status codes 301 (Moved Permanently) and 302 (Found) allow the request method to be changed from POST to GET. The specification also defines the corresponding 307 (Temporary Redirect) and 308 (Permanent Redirect) status codes that don’t allow the request method to be changed from POST to GET.

Now let’s look at the code for redirecting a post request to another post request:
```java
@PostMapping("/redirectPostToPost")
public ModelAndView redirectPostToPost(HttpServletRequest request) {
    request.setAttribute(
      View.RESPONSE_STATUS_ATTRIBUTE, HttpStatus.TEMPORARY_REDIRECT);
    return new ModelAndView("redirect:/redirectedPostToPost");
}

@PostMapping("/redirectedPostToPost")
public ModelAndView redirectedPostToPost() {
    return new ModelAndView("redirection");
}
```

__Now, let’s test the redirect of POST using the curl command:__
```
curl -L --verbose -X POST http://localhost:8080/spring-rest/redirectPostToPost
```

We are getting redirected to the destined location:
```
> POST /redirectedPostToPost HTTP/1.1
> Host: localhost:8080
> User-Agent: curl/7.49.0
> Accept: */*
>
< HTTP/1.1 200
< Content-Type: application/json;charset=UTF-8
< Transfer-Encoding: chunked
< Date: Tue, 08 Aug 2017 07:33:00 GMT

 {"id":1,"content":"redirect completed"}
```
