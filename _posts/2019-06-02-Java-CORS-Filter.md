---
layout:     post
title:      Java CORS Filter Example
subtitle:   Introduction to Java CORS Filter
date:       2019-06-02
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
    - Java
---

Cross-origin resource sharing (__CORS__) is a mechanism that allows JavaScript on a web page to make [AJAX](https://howtodoinjava.com/scripting/jquery/jquery-ajax-tutorial/) requests to another domain, different from the domain from where it originated. By default, such web requests are forbidden in browsers, and they will result into _same origin security policy_ errors. Using __Java CORS filter__, you may allow the webpage to make requests from other domains as well (known as _cross domain requests_).

> Read More : [Spring security CORS filter](https://howtodoinjava.com/spring5/webmvc/spring-mvc-cors-configuration/)

### 1. How CORS Filter Work?
CORS capability works by adding some specific HTTP headers that tell the browser that downloaded webpage should be allowed to make web requests to given/all domains. Also, you can add information to instruct browser to allow only certain HTTP methods (GET/PUT/POST/DELETE etc) on those domain URLs.

You will read a term ‘__preflight request__‘ in rest of the post, so let’s understand it first.
> A CORS preflight request is a CORS request that checks to see if the CORS protocol is understood by another domain. It is an OPTIONS request using two HTTP request headers: Access-Control-Request-Method and Access-Control-Request-Headers , and the Origin header.

A preflight request is automatically issued by a browser when needed; in normal cases, front-end developers don’t need to write such requests themselves. In response to a preflight request the resource indicates which methods and headers it is willing to handle and whether it supports credentials.

Now let’s go through CORS related HTTP headers to understand more.

#### 1.1. Response Headers
* __Access-Control-Allow-Origin__ : specifies the authorized domains to make cross-domain request. Use “*” as value if there is no restrictions.
* __Access-Control-Allow-Credentials__ : specifies if cross-domain requests can have authorization credentials or not.
* __Access-Control-Expose-Headers__ : indicates which headers are safe to expose.
* __Access-Control-Max-Age__ : indicates how long the results of a preflight request can be cached.
* __Access-Control-Allow-Methods__ : indicates the methods allowed when accessing the resource.
* __Access-Control-Allow-Headers__ : indicates which header field names can be used during the actual request.

#### 1.2. Request Headers
* __Origin__ : indicates where the cross-origin actual request or preflight request originates from.
* __Access-Control-Request-Method__ : used when issuing a preflight request to let the server know what HTTP method will be used in actual request.
* __Access-Control-Request-Headers__ : used when issuing a preflight request to let the server know what HTTP headers will be used in actual request.

### 2. Java CORS Filter Example
Now let’s a very basic implementation of CORS filter which can be added to any web application.
```java
import java.io.IOException;
import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.annotation.WebFilter;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

/**
 * Servlet Filter implementation class CORSFilter
 */
// Enable it for Servlet 3.x implementations
/* @ WebFilter(asyncSupported = true, urlPatterns = { "/*" }) */
public class CORSFilter implements Filter {

    /**
     * Default constructor.
     */
    public CORSFilter() {
        // TODO Auto-generated constructor stub
    }

    /**
     * @see Filter#destroy()
     */
    public void destroy() {
        // TODO Auto-generated method stub
    }

    /**
     * @see Filter#doFilter(ServletRequest, ServletResponse, FilterChain)
     */
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain chain)
            throws IOException, ServletException {

        HttpServletRequest request = (HttpServletRequest) servletRequest;
        System.out.println("CORSFilter HTTP Request: " + request.getMethod());

        // Authorize (allow) all domains to consume the content
        ((HttpServletResponse) servletResponse).addHeader("Access-Control-Allow-Origin", "*");
        ((HttpServletResponse) servletResponse).addHeader("Access-Control-Allow-Methods","GET, OPTIONS, HEAD, PUT, POST");

        HttpServletResponse resp = (HttpServletResponse) servletResponse;

        // For HTTP OPTIONS verb/method reply with ACCEPTED status code -- per CORS handshake
        if (request.getMethod().equals("OPTIONS")) {
            resp.setStatus(HttpServletResponse.SC_ACCEPTED);
            return;
        }

        // pass the request along the filter chain
        chain.doFilter(request, servletResponse);
    }

    /**
     * @see Filter#init(FilterConfig)
     */
    public void init(FilterConfig fConfig) throws ServletException {
        // TODO Auto-generated method stub
    }

}
```

Now register this filter in __web.xml__.
```xml
<filter>
    <filter-name>CorsFilter</filter-name>
    <filter-class>com.howtodoinjava.examples.cors.CORSFilter</filter-class>
</filter>

<filter-mapping>
    <filter-name>CorsFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

That’s all regarding using CORS filters in java web applications.
