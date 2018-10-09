---
layout:     post
title:      Http Message Converters with The Spring
subtitle:   Introduction to Http Message Converters in Spring
date:       2019-06-02
author:     Mr.Humorous 🥘
header-img: img/springFramework/header.jpg
catalog: true
tags:
    - Spring MVC
    - Spring Framework
---

## Introduction
To put in laymen terms, _message converters are to marshal and unmarshal object_ in a different format (like JSON, XML etc.). Spring MVC uses the _HttpMessageConverter_ interface to convert HTTP requests and responses. _Spring MVC_ provides some out of the box default configurations for the converters.

### 1. MVC Configurations
To enable support for the __Http message converters with the Spring MVC__, we need to enable _Spring MVC_ support either using Java config or the traditional XML based configuration.
```java
@Configuration
@EnableWebMvc
public class WebMVCConfig implements WebMvcConfigurer {
// Implement configuration methods...
}
```

For the XML, used <mvc:annotation-driven/>
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <mvc:annotation-driven/>
</beans>
```

Above configurations help Spring MVC in:
* Register and enable a number of core/infrastructure beans (e.g HandlerMapper, ViewResolver, ThemeResolver, LocalResolver etc. Beans).
* Configure and enable different message converter (e.g Json, XML etc.)

### 2. The Default Message Converters
By default, the following __HttpMessageConverters__ are loaded by Spring:
* __ByteArrayHttpMessageConverter__ – that can read and write byte arrays
* __ResourceHttpMessageConverter__ -can read/write Resources and supports byte range request
* __SourceHttpMessageConverter__ – can read and write Source objects.
* __StringHttpMessageConverter__ – read and write strings
* __FormHttpMessageConverter__ – read and write ‘normal’ HTML forms .

Spring provides other sets of message converters, but they will only be registered when the related library is in the classpath, here are some of the popular converters from the list:
* MappingJacksonHttpMessageConverter –  For converting JSON
* Jaxb2RootElementHttpMessageConverter –  To convert Java objects to/from XML
* MappingJackson2HttpMessageConverter – To convert JSON.

Read [HttpMessageConverter](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/http/converter/HttpMessageConverter.html) for more detail.

### 3. Spring MVC Content Negotiation
_Http message converters_ work in close relation with the content negotiation.Before getting into more details, let’s discuss how content negotiation works in Spring.
1. With the new request, Spring use _"Accept"_ header to find what type of representation required on the client side.
2. __Content-Type__: This header tells about the media type of the body of the request.
3. This header helps server or the client to select the correct approach to handle and eventually parse the data by selecting right converter.
4. Based on this, Spring will try to find a converter which can handle and parse this request.

Let’s take a simple example to understand this more clearly.
* Client or browser send GET request to /customer resource with Accept header as application/json.
* Based on the _Accept_ header, Spring will choose appropriate converter to parse the incoming request (In our example, it will choose JSON message converter).
* If no right converter is found, Spring will throw the exception.

Read [Content Negotiation](https://www.javadevjournal.com/spring/rest/content-negotiation/) for more detail.Let’s take a deeper look at __@RequestBody__ and __@ResponseBody__ annotation to see how they work with Http message converters the Spring.

#### 3.1 @RequestBody Annotation
Annotation indicating a method parameter should be bound to the body of the web request. It works with _"Content-Type"_ header passed by the client. Spring use this header value to find the right converter.
```java
@PostMapping(value = "/greeting")
public @ResponseBody void greeting(@RequestBody Customer customer){
   //create customer
}
```

Use any _REST client_ or curl to execute code, we will get 200 OK response.
```bash
curl -i -X POST -H "Content-Type: application/json" -d '{"id":"14","name":"Java Dev Journal"}' http://localhost:8080/greeting
HTTP/1.1 200
Content-Length: 0
Date: Tue, 27 Mar 2018 04:28:23 GMT
```

#### 3.2 @ResponseBody Annotation
@ResponseBody is used on the argument of a Controller method.This annotation tells Spring that the return value of the method serialized directly to the body of the _HTTP_ Response.

@ResponseBody annotation work with _"Accept"_ header passed by the client to decide what type of representation required on the client side.
```java
@GetMapping(value = "/greeting")
public @ResponseBody Customer greeting(){
  Customer customer = new Customer(14, "Java Dev Journal");
  return customer;
}
```

Output
```bash
curl --header "Accept: application/json" http://localhost:8080/greeting
{
   "id":14,
   "name":"Java Dev Journal"
}
```

### 4. Custom Converter
To create a custom message converter in Spring, we should override configureMessageConverters() by extending WebMvcConfigurerAdapter class.
```java
@Configuration
public class WebConfig implements WebMvcConfigurerAdapter {

 @Override
 public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {

   MarshallingHttpMessageConverter xmlConverter =new MarshallingHttpMessageConverter();
   XStreamMarshaller xstream = new XStreamMarshaller();
   xmlConverter.setMarshaller(xstream);
   xmlConverter.setUnmarshaller(xstream);
   converters.add(xmlConverter);
 }
}
```
> We need Xtream library in the classpath

Let’s check what we did in this example:
* We are overriding _configureMessageConverters()_ method under WebMvcConfigurerAdapter to define our own custom converter.
* Created instance of _MarshallingHttpMessageConverter_ to read and write XML using _Spring’s Marshaller_ and _Unmarshaller_.
* We created XStreamMarshaller instance and set Marshaller and Unmarshaller for the MarshallingHttpMessageConverter.
* Finally, we added this converter to the list of converters.

Let’s see our custom converter in action:
```bash
curl --header "Accept: application/json" http://localhost:8080/greeting
curl --header "Accept: application/xml" http://localhost:8080/greeting
```

Output
```json
{
   "id":14,
   "name":"Java Dev Journal"
}
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<customer>
   <id>14</id>
   <name>Java Dev Journal</name>
</customer>
```
