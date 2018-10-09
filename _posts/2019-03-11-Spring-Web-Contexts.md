---
layout:     post
title:      Spring Web Contexts
subtitle:   Understand how to organize application contexts
date:       2019-03-11
author:     Mr.Humorous 🥘
header-img: img/springFramework/header.jpg
catalog: true
tags:
    - Spring Framework
    - Spring MVC
---

## 1. Introduction
When using Spring in a web application, we have several options for organizing the application contexts that wire it all up.

## 2. The Root Web Application Context
__Every Spring webapp has an associated application context that is tied to its lifecycle: the root web application context.__

This is an old feature that predates Spring Web MVC, so it’s not tied specifically to any web framework technology.

The context is started when the application starts, and it’s destroyed when it stops, thanks to a servlet context listener. The most common types of contexts can also be refreshed at runtime, although not all _ApplicationContext_ implementations have this capability.

The context in a web application is always an instance of _WebApplicationContext_. That’s an interface extending _ApplicationContext_ with a contract for accessing the _ServletContext_.

Anyway, applications usually should not be concerned about those implementation details: __the root web application context is simply a centralized place to define shared beans.__

### 2.1. The ContextLoaderListener
The root web application context described in the previous section is managed by a listener of class _org.springframework.web.context.ContextLoaderListener_, which is part of the _spring-web_ module.

__By default, the listener will load an XML application context from */WEB-INF/applicationContext.xml*.__ However, those defaults can be changed. We can use Java annotations instead of XML, for example.

We can configure this listener either in the webapp descriptor (_web.xml_ file) or programmatically in Servlet 3.x environments.

In the following sections, we’ll look at each of these options in detail.

### 2.2. Using web.xml and an XML Application Context
When using web.xml, we configure the listener as usual:
```xml
<listener>
    <listener-class>
        org.springframework.web.context.ContextLoaderListener
    </listener-class>
</listener>
```

We can specify an alternate location of the XML context configuration with the _contextConfigLocation parameter_:
```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/rootApplicationContext.xml</param-value>
</context-param>
```

Or more than one location, separated by commas:
```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/context1.xml, /WEB-INF/context2.xml</param-value>
</context-param>
```

We can even use patterns:
```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/*-context.xml</param-value>
</context-param>
```

In any case, __only one context is defined__, by combining all the bean definitions loaded from the specified locations.

### 2.3. Using web.xml and a Java Application Context
We can also specify other types of contexts besides the default XML-based one. Let’s see, for example, how to use Java annotations configuration instead.

We use the _contextClass_ parameter to tell the listener which type of context to instantiate:
```xml
<context-param>
    <param-name>contextClass</param-name>
    <param-value>
        org.springframework.web.context.support.AnnotationConfigWebApplicationContext
    </param-value>
</context-param>
```

__Every type of context may have a default configuration location. In our case, the _AnnotationConfigWebApplicationContext_ does not have one, so we have to provide it.__

We can thus list one or more annotated classes:
```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>
        com.baeldung.contexts.config.RootApplicationConfig,
        com.baeldung.contexts.config.NormalWebAppConfig
    </param-value>
</context-param>
```

Or we can tell the context to scan one or more packages:
```xml
<context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>org.baeldung.bean.config</param-value>
</context-param>
```

And, of course, we can mix and match the two options.

### 2.4. Programmatic Configuration with Servlet 3.x
__Version 3 of the Servlet API has made configuration through the web.xml file completely optional__. Libraries can provide their web fragments, which are pieces of XML configuration that can register listeners, filters, servlets and so on.

Also, users have access to an API that allows defining programmatically every element of a servlet-based application.

The _spring-web module_ makes use of these features and offers its API to register components of the application when it starts.

Spring scans the application’s classpath for instances of the _org.springframework.web.WebApplicationInitializer_ class. This is an interface with a single method, _void onStartup(ServletContext servletContext) throws ServletException_, that’s invoked upon application startup.

Let’s now look at how we can use this facility to create the same types of root web application contexts that we’ve seen earlier.

### 2.5. Using Servlet 3.x and an XML Application Context
Let’s start with an XML context, just like in Section 2.2.

We’ll implement the aforementioned _onStartup_ method:
```java
public class ApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext servletContext)
      throws ServletException {
        //...
    }
}
```

__Let’s break the implementation down line by line.__

We first create a root context. Since we want to use XML, it has to be an XML-based application context, and since we’re in a web environment, it has to implement WebApplicationContext as well.

The first line, thus, is the explicit version of the _contextClass_ parameter that we’ve encountered earlier, with which we decide which specific context implementation to use:
```java
XmlWebApplicationContext rootContext = new XmlWebApplicationContext();
```

Then, in the second line, we tell the context where to load its bean definitions from. Again, setConfigLocations is the programmatic analogous of the contextConfigLocation parameter in web.xml:
```java
rootContext.setConfigLocations("/WEB-INF/rootApplicationContext.xml");
```

Finally, we create a _ContextLoaderListener_ with the root context and register it with the servlet container. As we can see, _ContextLoaderListener_ has an appropriate constructor that takes a _WebApplicationContext_ and makes it available to the application:
```java
servletContext.addListener(new ContextLoaderListener(rootContext));
```

### 2.6. Using Servlet 3.x and a Java Application Context
If we want to use an annotation-based context, we could change the code snippet in the previous section to make it instantiate an _AnnotationConfigWebApplicationContext_ instead.

However, let’s see a more specialized approach to obtain the same result.

__The WebApplicationInitializer class that we’ve seen earlier is a general-purpose interface.__ It turns out that Spring provides a few more specific implementations, including an abstract class called _AbstractContextLoaderInitializer_.

Its job, as the name implies, is to create a _ContextLoaderListener_ and register it with the servlet container.

We only have to tell it how to build the root context:
```java
public class AnnotationsBasedApplicationInitializer
  extends AbstractContextLoaderInitializer {

    @Override
    protected WebApplicationContext createRootApplicationContext() {
        AnnotationConfigWebApplicationContext rootContext
          = new AnnotationConfigWebApplicationContext();
        rootContext.register(RootApplicationConfig.class);
        return rootContext;
    }
}
```

Here we can see that we no longer need to register the _ContextLoaderListener_, which saves us from a little bit of boilerplate code.

Note also the use of the _register_ method that is specific to _AnnotationConfigWebApplicationContext_ instead of the more generic _setConfigLocations_: by invoking it, we can register individual _@Configuration_ annotated classes with the context, thus avoiding package scanning.

## 3. Dispatcher Servlet Contexts
Let’s now focus on another type of application context. This time, we’ll be referring to a feature which is specific to Spring MVC, rather than part of Spring’s generic web application support.

__Spring MVC applications have at least one Dispatcher Servlet configured__ (but possibly more than one, we’ll talk about that case later). This is the servlet that receives incoming requests, dispatches them to the appropriate controller method, and returns the view.

__Each *DispatcherServlet* has an associated application context__. Beans defined in such contexts configure the servlet and define MVC objects like controllers and view resolvers.

Let’s see how to configure the servlet’s context first. We’ll look at some in-depth details later.

### 3.1. Using web.xml and an XML Application Context
_DispatcherServlet_ is typically declared in web.xml with a name and a mapping:
```xml
<servlet>
    <servlet-name>normal-webapp</servlet-name>
    <servlet-class>
        org.springframework.web.servlet.DispatcherServlet
    </servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>normal-webapp</servlet-name>
    <url-pattern>/api/*</url-pattern>
</servlet-mapping>
```

If not otherwise specified, the name of the servlet is used to determine the XML file to load. In our example, we’ll use the file _WEB-INF/normal-webapp-servlet.xml_.

We can also specify one or more paths to XML files, in a similar fashion to _ContextLoaderListener_:
```xml
<servlet>
    ...
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/normal/*.xml</param-value>
    </init-param>
</servlet>
```

### 3.2. Using web.xml and a Java Application Context
When we want to use a different type of context we proceed like with _ContextLoaderListener_, again. That is, we specify a _contextClass_ parameter along with a suitable _contextConfigLocation_:
```xml
<servlet>
    <servlet-name>normal-webapp-annotations</servlet-name>
    <servlet-class>
        org.springframework.web.servlet.DispatcherServlet
    </servlet-class>
    <init-param>
        <param-name>contextClass</param-name>
        <param-value>
            org.springframework.web.context.support.AnnotationConfigWebApplicationContext
        </param-value>
    </init-param>
    <init-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>com.baeldung.contexts.config.NormalWebAppConfig</param-value>
    </init-param>
    <load-on-startup>1</load-on-startup>
</servlet>
```

### 3.3. Using Servlet 3.x and an XML Application Context
__Again, we’ll look at two different methods for programmatically declaring a *DispatcherServlet*, and we’ll apply one to an XML context and the other to a Java context.__

So, let’s start with a generic _WebApplicationInitializer_ and an XML application context.

As we’ve seen previously, we have to implement the _onStartup_ method. However, this time we’ll create and register a dispatcher servlet, too:
```java
XmlWebApplicationContext normalWebAppContext = new XmlWebApplicationContext();
normalWebAppContext.setConfigLocation("/WEB-INF/normal-webapp-servlet.xml");
ServletRegistration.Dynamic normal
  = servletContext.addServlet("normal-webapp",
    new DispatcherServlet(normalWebAppContext));
normal.setLoadOnStartup(1);
normal.addMapping("/api/*");
```

We can easily draw a parallel between the above code and the equivalent _web.xml_ configuration elements.

### 3.4. Using Servlet 3.x and a Java Application Context
This time, we’ll configure an annotations-based context using a specialized implementation of _WebApplicationInitializer: AbstractDispatcherServletInitializer_.

That’s an abstract class that, besides creating a root web application context as previously seen, allows us to register one dispatcher servlet with minimum boilerplate:
```java
@Override
protected WebApplicationContext createServletApplicationContext() {

    AnnotationConfigWebApplicationContext secureWebAppContext
      = new AnnotationConfigWebApplicationContext();
    secureWebAppContext.register(SecureWebAppConfig.class);
    return secureWebAppContext;
}

@Override
protected String[] getServletMappings() {
    return new String[] { "/s/api/*" };
}
```

Here we can see a method for creating the context associated with the servlet, exactly like we’ve seen before for the root context. Also, we have a method to specify the servlet’s mappings, as in _web.xml_.

## 4. Parent and Child Contexts
So far, we’ve seen two major types of contexts: the root web application context and the dispatcher servlet contexts. Then, we might have a question: __are those contexts related__?

It turns out that yes, they are. In fact, __the root context is the parent of every dispatcher servlet context__. Thus, beans defined in the root web application context are visible to each dispatcher servlet context, but not vice versa.

__So, typically, the root context is used to define service beans, while the dispatcher context contains those beans that are specifically related to MVC.__

Note that we’ve also seen ways to create the dispatcher servlet context programmatically. If we manually set its parent, then Spring does not override our decision, and this section no longer applies.

In simpler MVC applications, it’s sufficient to have a single context associated to the only one dispatcher servlet. There’s no need for overly complex solutions!

Still, the parent-child relationship becomes useful when we have multiple dispatcher servlets configured. But when should we bother to have more than one?

In general, __we declare multiple dispatcher servlets when we need multiple sets of MVC configuration.__ For example, we may have a REST API alongside a traditional MVC application or an unsecured and a secure section of a website:
![Contexts Hierarchy](/img/springFramework/springWebContexts/contextsHierarchy.png)

Note: when we extend _AbstractDispatcherServletInitializer_ (see section 3.4), we register both a root web application context and a single dispatcher servlet.

So, if we want more than one servlet, we need multiple _AbstractDispatcherServletInitializer_ implementations. However, we can only define one root context, or the application won’t start.

Fortunately, the _createRootApplicationContext_ method can return _null_. Thus, we can have one _AbstractContextLoaderInitializer_ and many _AbstractDispatcherServletInitializer_ implementations that don’t create a root context. In such a scenario, it is advisable to order the initializers with _@Order_ explicitly.

Also, note that _AbstractDispatcherServletInitializer_ registers the servlet under a given name (_dispatcher_) and, of course, we cannot have multiple servlets with the same name. So, we need to override _getServletName_:
```java
@Override
protected String getServletName() {
    return "another-dispatcher";
}
```

## 5. A Parent and Child Context Example
Suppose that we have two areas of our application, for example a public one which is world accessible and a secured one, with different MVC configurations. Here, we’ll just define two controllers that output a different message.

Also, suppose that some of the controllers need a service that holds significant resources; a ubiquitous case is persistence. Then, we’ll want to instantiate that service only once, to avoid doubling its resource usage, and because we believe in the Don’t Repeat Yourself principle!

Let’s now proceed with the example.

### 5.1. The Shared Service
In our hello world example, we settled for a simpler greeter service instead of persistence:
```java
package com.baeldung.contexts.services;

@Service
public class GreeterService {
    @Resource
    private Greeting greeting;

    public String greet() {
        return greeting.getMessage();
    }
}
```

We’ll declare the service in the root web application context, using component scanning:
```java
@Configuration
@ComponentScan(basePackages = { "com.baeldung.contexts.services" })
public class RootApplicationConfig {
    //...
}
```

We might prefer XML instead:
```xml
<context:component-scan base-package="com.baeldung.contexts.services" />
```

### 5.2. The Controllers
Let’s define two simple controllers which use the service and output a greeting:
```java
package com.baeldung.contexts.normal;

@Controller
public class HelloWorldController {

    @Autowired
    private GreeterService greeterService;

    @RequestMapping(path = "/welcome")
    public ModelAndView helloWorld() {
        String message = "<h3>Normal " + greeterService.greet() + "</h3>";
        return new ModelAndView("welcome", "message", message);
    }
}

//"Secure" Controller
package com.baeldung.contexts.secure;

String message = "<h3>Secure " + greeterService.greet() + "</h3>";
```

As we can see, the controllers lie in two different packages and print different messages: one says “normal”, the other “secure”.

### 5.3. The Dispatcher Servlet Contexts
As we said earlier, we’re going to have two different dispatcher servlet contexts, one for each controller. So, let’s define them, in Java:
```java
//Normal context
@Configuration
@EnableWebMvc
@ComponentScan(basePackages = { "com.baeldung.contexts.normal" })
public class NormalWebAppConfig implements WebMvcConfigurer {
    //...
}

//"Secure" context
@Configuration
@EnableWebMvc
@ComponentScan(basePackages = { "com.baeldung.contexts.secure" })
public class SecureWebAppConfig implements WebMvcConfigurer {
    //...
}
```

Or, if we prefer, in XML:
```xml
<!-- normal-webapp-servlet.xml -->
<context:component-scan base-package="com.baeldung.contexts.normal" />

<!-- secure-webapp-servlet.xml -->
<context:component-scan base-package="com.baeldung.contexts.secure" />
```

### 5.4. Putting It All Together
Now that we have all the pieces, we just need to tell Spring to wire them up. Recall that we need to load the root context and define the two dispatcher servlets. Although we’ve seen multiple ways to do that, we’ll now focus on two scenarios, a Java one and an XML one. _Let’s start with Java_.

We’ll define an _AbstractContextLoaderInitializer_ to load the root context:
```java
@Override
protected WebApplicationContext createRootApplicationContext() {
    AnnotationConfigWebApplicationContext rootContext
      = new AnnotationConfigWebApplicationContext();
    rootContext.register(RootApplicationConfig.class);
    return rootContext;
}
```

Then, we need to create the two servlets, thus we’ll define two subclasses of _AbstractDispatcherServletInitializer_. First, the “normal” one:
```java
@Override
protected WebApplicationContext createServletApplicationContext() {
    AnnotationConfigWebApplicationContext normalWebAppContext
      = new AnnotationConfigWebApplicationContext();
    normalWebAppContext.register(NormalWebAppConfig.class);
    return normalWebAppContext;
}

@Override
protected String[] getServletMappings() {
    return new String[] { "/api/*" };
}

@Override
protected String getServletName() {
    return "normal-dispatcher";
}
```

Then, the “secure” one, which loads a different context and is mapped to a different path:
```java
@Override
protected WebApplicationContext createServletApplicationContext() {
    AnnotationConfigWebApplicationContext secureWebAppContext
      = new AnnotationConfigWebApplicationContext();
    secureWebAppContext.register(SecureWebAppConfig.class);
    return secureWebAppContext;
}

@Override
protected String[] getServletMappings() {
    return new String[] { "/s/api/*" };
}

@Override
protected String getServletName() {
    return "secure-dispatcher";
}
```

And we’re done! We’ve just applied what we touched in previous sections.

__We can do the same with web.xml__, again just by combining the pieces we’ve discussed so far.

Define a root application context:
```xml
<listener>
    <listener-class>
        org.springframework.web.context.ContextLoaderListener
    </listener-class>
</listener>
```

A “normal” dispatcher context:
```xml
<servlet>
    <servlet-name>normal-webapp</servlet-name>
    <servlet-class>
        org.springframework.web.servlet.DispatcherServlet
    </servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>normal-webapp</servlet-name>
    <url-pattern>/api/*</url-pattern>
</servlet-mapping>
```

And, finally, a “secure” context:
```xml
<servlet>
    <servlet-name>secure-webapp</servlet-name>
    <servlet-class>
        org.springframework.web.servlet.DispatcherServlet
    </servlet-class>
    <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
    <servlet-name>secure-webapp</servlet-name>
    <url-pattern>/s/api/*</url-pattern>
</servlet-mapping>
```

## 6. Combining Multiple Contexts
There are other ways than parent-child to combine multiple configuration locations, __to split big contexts and better separate different concerns__. We’ve seen one example already: when we specify _contextConfigLocation_ with multiple paths or packages, Spring builds a single context by combining all the bean definitions, as if they were written in a single XML file or Java class, in order.

However, we can achieve a similar effect with other means and even use different approaches together. Let’s examine our options.

One possibility is component scanning, which we explain [in another article](https://www.baeldung.com/spring-bean-annotations#scanning).

### 6.1. Importing a Context Into Another
Alternatively, we can have a context definition import another one. Depending on the scenario, we have different kinds of imports.

Importing a _@Configuration_ class in Java:
```java
@Configuration
@Import(SomeOtherConfiguration.class)
public class Config { ... }
```

Loading some other type of resource, for example, an XML context definition, in Java:
```java
@Configuration
@ImportResource("classpath:basicConfigForPropertiesTwo.xml")
public class Config { ... }
```

Finally, including an XML file in another one:
```xml
<import resource="greeting.xml" />
```

Thus, we have many ways to organize the services, components, controllers, etc., that collaborate to create our awesome application. And the nice thing is that IDEs understand them all!

## 7. Spring Boot Web Applications
__Spring Boot automatically configures the components of the application__, so, generally, there is less need to think about how to organize them.

Still, under the hood, Boot uses Spring features, including those that we’ve seen so far. Let’s see a couple of noteworthy differences.

Spring Boot web applications running in an embedded container [don’t run any WebApplicationInitializer by design](https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/#boot-features-embedded-container-context-initializer).

Should it be necessary, we can write the same logic in a _SpringBootServletInitializer_ or a _ServletContextInitializer_ instead, depending on the chosen deployment strategy.

However, for adding servlets, filters, and listeners as seen in this article, it is not necessary to do so. __In fact, Spring Boot automatically registers every servlet-related bean to the container__:
```java
@Bean
public Servlet myServlet() { ... }
```

The objects so defined are mapped according to conventions: filters are automatically mapped to /*, that is, to every request. If we register a single servlet, it is mapped to /, otherwise, each servlet is mapped to its bean name.

If the above conventions don’t work for us, we can define a _FilterRegistrationBean, ServletRegistrationBean, or ServletListenerRegistrationBean_ instead. Those classes allow us to control the fine aspects of the registration.

## 8. Conclusions
In this article, we’ve given an in-depth view of the various options available to structure and organize a Spring web application.

We’ve left out some features, notably the [support for a shared context in enterprise applications](https://spring.io/blog/2007/06/11/using-a-shared-parent-application-context-in-a-multi-war-spring-application/), which, at the time of writing, is still [missing from Spring 5](https://jira.spring.io/browse/SPR-16258).

The implementation of all these examples and code snippets can be found in [the GitHub project](https://github.com/eugenp/tutorials/tree/master/spring-all) – this is a Maven project, so it should be easy to import and run as is.
