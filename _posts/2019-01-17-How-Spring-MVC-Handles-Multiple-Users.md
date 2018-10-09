---
layout:     post
title:      How Spring MVC handles multiple users
subtitle:   Understanding Spring bean scopes
date:       2019-01-17
author:     Mr.Humorous 🥘
header-img: img/springFramework/header.jpg
catalog: true
tags:
    - Spring Framework
    - Spring MVC
---

## 1. Problem Statement
I have a spring web app. Now I autowired the model in controller. Based on url matching it calls respective method. all my methods are singleton.

Now when two users are opening app at same time spring is able to run them parallelly and give results to them. I didnt understand how can it do this. i mean as the bean is singleton it has to either the wait till the bean is not used or overwrite the data in the bean. But spring is working correctly. Can someone explain this behaviour with some analogy.

To explain my question clearly below is a piece of code:

My default controller is simple one:
```java
@Autowired
private AppModel aModel;
public AppModel getModel(){
    return aModel;
}
public void setModel(AppModel aModel){
    this.aModel = aModel;
}

@RequestMapping(method = RequestMethod.GET)
public ModelAndView defaultGetter(HttpServletRequest request,
        HttpServletResponse response) throws Exception {
    ModelAndView mav = new ModelAndView(getViewName());
    mav.addObject("model", aModel);
    Runtime.getRuntime().gc();
    return mav;
}
```

Also can some one tell me when two clients open the app will two seperate models get generated when i use @autowired . If only one model bean exists for all clients then say the request from client 1 came in and it take me 30 sec to get results back. Now if second client sends request in 3rd sec then will the first clients request gets overwritten?

I think I am getting confused. Can some one clarify how this magic is happening?

## 2. Explanation
Every web request generates a new thread as [explained in this thread](https://stackoverflow.com/questions/7457190/how-threads-allocated-to-handle-servlet-request).

Spring manages different scopes (prototype, request, session, singleton). If two simultaneous requests access a singleton bean, then the bean must be stateless (or at least synchronized to avoid problems). If you access a bean in scope request, then a new instance will be generated per request. Spring manages this for you but you have to be careful and use the correct scope for your beans. Typically, your controller is a singleton but the `AppModel` has to be of scope `request`, otherwise you will have problems with two simultaneous requests. [This thread could also help you](https://stackoverflow.com/questions/11139571/scope-of-a-spring-controller-and-its-instance-variables).

About your last question "how this magic is happening?", the answer is "aspect/proxy". Spring create proxy classes. You can imagine that Spring will create a proxy to your `AppModel` class. As soon as you try to access it in the controller, Spring forwards the method call to the right instance.
