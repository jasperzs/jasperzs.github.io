---
layout:     post
title:      Spring CORS - Spring @CrossOrigin example
subtitle:   Introduction to Spring @CrossOrigin annotation
date:       2019-06-02
author:     Mr.Humorous 🥘
header-img: img/springFramework/header.jpg
catalog: true
tags:
    - Spring MVC
    - Spring Framework
---

__CORS__ (_Cross-origin resource sharing_) allows a webpage to request additional resources into browser from other domains e.g. fonts, CSS or static images from CDNs. CORS helps in serving web content from multiple domains into browsers who usually have the [same-origin](https://en.wikipedia.org/wiki/Same-origin_policy) security policy.

In this example, we will learn to enable __Spring CORS__ support in [Spring MVC](https://howtodoinjava.com/spring-mvc-tutorial/) application at method level and global level.

### 1. Spring CORS – Method level with @CrossOrigin
Spring MVC provides [@CrossOrigin](https://docs.spring.io/spring/docs/current/javadoc-api/org/springframework/web/bind/annotation/CrossOrigin.html) annotation. This annotation marks the annotated method or type as permitting cross origin requests.

#### 1.1. Spring CORS allow all
By default, __@CrossOrigin__ allows all origins, all headers, the [HTTP methods](https://restfulapi.net/http-methods/) specified in the [@RequestMapping](https://howtodoinjava.com/spring5/webmvc/controller-getmapping-postmapping/) annotation and a __maxAge__ of 30 minutes.

You can override default CORS settings by giving value to annotation attributes :
<table>
  <tbody>
    <tr>
      <th align="center">ATTRIBUTE</th>
      <th align="center">DESCRIPTION</th>
    </tr>
    <tr>
      <td style="text-align:center;vertical-align:middle">origins</td>
      <td align="left">
        List of allowed origins. It’s value is placed in the Access-Control-Allow-Origin header of both the pre-flight response and the actual response.
        <ul>
          <li>
            * – means that all origins are allowed.
          </li>
          <li>
            If undefined, all origins are allowed.
          </li>
        </ul>
      </td>
    </tr>
    <tr>
      <td style="text-align:center;vertical-align:middle">allowedHeaders</td>
      <td align="left">
        List of request headers that can be used during the actual request. Value is used in preflight’s response header Access-Control-Allow-Headers.
        <ul>
          <li>
            * – means that all headers requested by the client are allowed.
          </li>
          <li>
            If undefined, all requested headers are allowed.
          </li>
        </ul>
       </td>
    </tr>
    <tr>
      <td style="text-align:center;vertical-align:middle">methods</td>
      <td align="left">
        List of supported HTTP request methods. If undefined, methods defined by RequestMapping annotation are used.
        <ul>
          <li>
            * – means that all headers requested by the client are allowed.
          </li>
          <li>
            If undefined, all requested headers are allowed.
          </li>
        </ul>
       </td>
    </tr>
    <tr>
      <td style="text-align:center;vertical-align:middle">exposedHeaders</td>
      <td align="left">
        List of response headers that the browser will allow the client to access. Value is set in actual response header Access-Control-Expose-Headers.
        <ul>
          <li>
            If undefined, an empty exposed header list is used.
          </li>
        </ul>
       </td>
    </tr>
    <tr>
      <td style="text-align:center;vertical-align:middle">allowCredentials</td>
      <td align="left">
        It determine whether browser should include any cookies associated with the request.
        <ul>
          <li>
            false – cookies should not included.
          </li>
          <li>
            "" (empty string) – means _undefined_.
          </li>
          <li>
            true – pre-flight response will include the header Access-Control-Allow-Credentials with value set to true.
          </li>
          <li>
            If undefined, credentials are allowed.
          </li>
        </ul>
       </td>
    </tr>
    <tr>
      <td style="text-align:center;vertical-align:middle">maxAge</td>
      <td align="left">
        maximum age (in seconds) of the cache duration for pre-flight responses. Value is set in header Access-Control-Max-Age.
        <ul>
          <li>
            If undefined, max age is set to 1800 seconds (30 minutes).
          </li>
        </ul>
       </td>
    </tr>
  </tbody>
</table>

#### 1.2. @CrossOrigin at Class/Controller Level
```java
@CrossOrigin(origins = "*", allowedHeaders = "*")
@Controller
public class HomeController
{
    @GetMapping(path="/")
    public String homeInit(Model model) {
        return "home";
    }
}
```

> Read More – [Spring 5 MVC Example](https://howtodoinjava.com/spring5/webmvc/spring5-mvc-hibernate5-example/)

#### 1.3. @CrossOrigin at Method Level
```java
@Controller
public class HomeController
{
    @CrossOrigin(origins = "*", allowedHeaders = "*")
    @GetMapping(path="/")
    public String homeInit(Model model) {
        return "home";
    }
}
```

#### 1.4. @CrossOrigin Overridden at Method Level
homeInit() method will be accessible only from domain http://example.com. Rest other methods in HomeController will be accessible from all domains.
```java
@Controller
@CrossOrigin(origins = "*", allowedHeaders = "*")
public class HomeController
{
    @CrossOrigin(origins = "http://example.com")
    @GetMapping(path="/")
    public String homeInit(Model model) {
        return "home";
    }
}
```

### 2. Spring CORS – Global CORS configuration

#### 2.1. Spring MVC CORS with WebMvcConfigurerAdapter
To enable CORS for the whole application, use WebMvcConfigurerAdapter to add CorsRegistry.
```java
@Configuration
@EnableWebMvc
public class CorsConfiguration extends WebMvcConfigurerAdapter
{
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/**")
                .allowedMethods("GET", "POST");
    }
}
```

#### 2.2. Spring Boot CORS with WebMvcConfigurer
In spring boot application, it is recommended to just declare a WebMvcConfigurer bean.
```java
@Configuration
public class CorsConfiguration
{
    @Bean
    public WebMvcConfigurer corsConfigurer()
    {
        return new WebMvcConfigurerAdapter() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/**");
            }
        };
    }
}
```

#### 2.3. CORS with Spring Security
To enable CORS support through [Spring security](https://howtodoinjava.com/spring-security-tutorial/), configure CorsConfigurationSource bean and use HttpSecurity.cors() configuration.
```java
@EnableWebSecurity
public class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.cors().and()
            //other config
    }

    @Bean
    CorsConfigurationSource corsConfigurationSource()
    {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(Arrays.asList("https://example.com"));
        configuration.setAllowedMethods(Arrays.asList("GET","POST"));
        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }
}
```
