---
layout:     post
title:      Spring MVC and the @ModelAttribute Annotation
subtitle:   How to use the @ModelAttribute Annotation
date:       2018-12-18
author:     Mr.Humorous 🥘
header-img: img/springFramework/header.jpg
catalog: true
tags:
    - Spring Framework
    - Spring MVC
---

## 1. Overview

One of the most important Spring-MVC annotations is the `@ModelAttribute` annotation.

The `@ModelAttribute` is an annotation that binds a method parameter or method return value to a named model attribute and then exposes it to a web view.

The code of this simple tutorial is at [github](https://github.com/eugenp/tutorials/tree/master/spring-mvc-java).

## 2. The @ModelAttribute in Depth

### 2.1 At the Method Level

When the annotation is used at the method level it indicates the purpose of that method is to add one or more model attributes. Such methods support the same argument types as `@RequestMapping` methods but that cannot be mapped directly to requests.

Let’s have a look at a quick example here to start understanding how this works:
```java
@ModelAttribute
public void addAttributes(Model model) {
    model.addAttribute("msg", "Welcome to the Netherlands!");
}
```

In the example, we show a method that adds an attribute named msg to all models defined in the controller class.

In general, Spring-MVC will always make a call first to that method, before it calls any request handler methods. __That is, @ModelAttribute methods are invoked before the controller methods annotated with @RequestMapping are invoked__. The logic behind the sequence is that, the model object has to be created before any processing starts inside the controller methods.

It is also important that you annotate the respective class as `@ControllerAdvice`. Thus, you can add values in Model which will be identified as global. This actually means that for every request a default value exists, for every method in the response part.

### 2.2 As a Method Argument

When used as a method argument, it indicates the argument should be retrieved from the model. When not present, it should be first instantiated and then added to the model and once present in the model, the arguments fields should be populated from all request parameters that have matching names.

In the code snippet that follows the _employee_ model attribute is populated with data from a form submitted to the _addEmployee_ endpoint. Spring MVC does this behind the scenes before invoking the submit method:
```java
@RequestMapping(value = "/addEmployee", method = RequestMethod.POST)
public String submit(@ModelAttribute("employee") Employee employee) {
    // Code that uses the employee object

    return "employeeView";
}
```

Later on in this article we will see a complete example of how to use the employee object to populate the employeeView template.

So, it binds the form data with a bean. __The controller annotated with @RequestMapping can have custom class argument(s) annotated with @ModelAttribute__.

This is what is commonly known as data binding in Spring-MVC, a common mechanism that saves you from having to parse each form field individually.

## 3. Form Example

In this section we will provide the example referred to in the overview section: a very basic form that prompts a user (employee of a company, in our specific example), to enter some personal information (specifically _name_ and _id_). After the submission is completed and without any errors, the user expects to see the previously submitted data, displayed on another screen.

### 3.1 The View

Let’s first create a simple form with id and name fields:
```java
<form:form method="POST" action="/spring-mvc-java/addEmployee"
  modelAttribute="employee">
    <form:label path="name">Name</form:label>
    <form:input path="name" />

    <form:label path="id">Id</form:label>
    <form:input path="id" />

    <input type="submit" value="Submit" />
</form:form>
```

### 3.2 The Controller

Here is the controller class, where the logic for the afore-mentioned view is being implemented:
```java
@Controller
@ControllerAdvice
public class EmployeeController {

    private Map<Long, Employee> employeeMap = new HashMap<>();

    @RequestMapping(value = "/addEmployee", method = RequestMethod.POST)
    public String submit(
      @ModelAttribute("employee") Employee employee,
      BindingResult result, ModelMap model) {
        if (result.hasErrors()) {
            return "error";
        }
        model.addAttribute("name", employee.getName());
        model.addAttribute("id", employee.getId());

        employeeMap.put(employee.getId(), employee);

        return "employeeView";
    }

    @ModelAttribute
    public void addAttributes(Model model) {
        model.addAttribute("msg", "Welcome to the Netherlands!");
    }
}
```

In the _submit()_ method we have an _Employee_ object bound to our _View_. Can you see the power of this annotation? You can map your form fields to an object model as simply as that. In the method we are fetching values from the form and setting them to _ModelMap_.

__In the end we return employeeView, which means that the respective JSP file is going to be called as a View representative__.

Furthermore, there is also an _addAttributes()_ method. Its purpose is to add values in the _Model_ which will be identified globally. That is, a default value will be returned as a response for every request to every controller method. We also have to annotate the specific class as _@ControllerAdvice_.

### 3.3 The Model

As mentioned before, the _Model_ object is very simplistic and contains all that is required by the “front-end” attributes. Now, let’s have a look at an example:
```java
@XmlRootElement
public class Employee {

    private long id;
    private String name;

    public Employee(long id, String name) {
        this.id = id;
        this.name = name;
    }

    // standard getters and setters removed
}
```

### 3.4 Wrap Up

The _@ControllerAdvice_ assists a controller and in particular, _@ModelAttribute_ methods that apply to all _@RequestMapping_ methods. Of course, our _addAttributes()_ method will be the very first to run, prior to the rest of the _@RequestMapping_ methods.

Keeping that in mind and after both of _submit()_ and _addAttributes()_ are run, we could just refer to them in the View returned from the _Controller_ class, by mentioning their given name inside a dollarized curly-braces duo, like for example _${name}_.


### 3.5 Results View

Let’s now print what we received from the form:
```java
<h3>${msg}</h3>
Name : ${name}
ID : ${id}
```
