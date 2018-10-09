---
layout:     post
title:      Basics of Software Architecture Design in Java
subtitle:   Understanding Software Architecture Design
date:       2018-11-18
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
    - Java
    - Software Design
---

## 1. SOLID Principles
- ###### <ins>_S  <-->  SINGLE RESPONSIBILITY PRINCIPLE_</ins> ######
  + A software entity (class, method) should have only one reason to change
  + If a class or a method does more than one procedure, it is advisable to separate it into two distinct classes/methods. It can be done easily with the help of interfaces !!!
  + So if we have 2 reasons to change for a class -> we should split the functionality into two separate classes. __EACH CLASS WILL HANDLE A SINGLE RESPONSIBILITY !!!__
  + It leads to a low coupled design with less and lighter dependencies
- ###### <ins>_O  ->  OPEN/CLOSED PRINCIPLE_</ins> ######
  + What is the MOTIVATION behind SOLID principles?
    * An application should take care of the frequent changes that are done during the development and the maintaining phase of an application. For example: new functionalities have to be added !!!
    * Those changes in the existing code should be minimized. WHY? It's assumed that the existing code is already unit tested and changes in already written code might affect the existing functionality in an unwanted manner.
  + Open closed principle states that the design and writing of the code should be done in a way that new functionality should be added with minimum changes in the existing code. __WE SHOULD KEEP AS MUCH EXISTING CODE UNCHANGED AS POSSIBLE !!!__
  + Software entities should be open for extension, but closed for modification. We have to design every new module -> if we add a new behavior, we do not have to change the existing modules.
  + Closely related to the single responsibility principle !!!
  + A class should not extend another class explicitly. We should have a common interface. Why? Because we can change the classes at runtime due to the common interface !!!
	  * For example: we want to show a progress dialog but it can due to some download or loading of some music etc ... We want to decide at runtime why we want to show the dialog.
  + We can make sure this principle is not violated:
    * Strategy pattern
    * Template pattern
- ###### <ins>_L  ->  LISKOV SUBSTITUTION PRINCIPLE_</ins> ######
  + What is the motivation of Liskov principle?
    * We usually create class hierarchies during the application development. For example: we extend some classes creating some derived classes !!!
    * It would be great if the new derived classes would work as well without replacing the functionality of the classes.
    * Otherwise the new classes can produce undesired effects when they are used in existing program modules.
  + Child classes should never break the parent class type definition.
  + Subtypes must be substitutable for their base types ( derived types must be completely substitutable for their base types ).
  + We have to make sure there will be no problems using the subtype or the original class. Do not break functionality !!! We can call the methods anyway. We can solve it with the help of Template Pattern
  + It is not independent from Open Close Principle + Interface Segregation Principle.
  + The violation of Liskov principle is a latent violation of Open Closed Principle !!!
- ###### <ins>_I -> INTERFACE SEGREGATION PRINCIPLE_</ins> ######
  + What is the motivation?
    * We can have an abstraction of the system using interfaces `List<String> l = new ArrayList<>();`
    * Sometimes --> we want to implement that interface but just for the sake of some methods defined in by that interface. BUT we have to implement all the methods --> "fat interfaces"
  + It is about business logic to clients communication - it is not good if an interface contains lots of methods. We should separate them accordingly.
  + The interface-segregation principle (ISP) states that no client should be forced to depend on methods it does not use !!!
  + When we can, we should break our interfaces in many smaller ones, so they better satisfy the exact needs of our clients.
- ###### <ins>_D -> DEPENDENCY INVERSION PRINCIPLE_</ins> ######
  + What is the MOTIVATION?
    * When we develop a software or an application (__HIGH LEVEL MODULES__ <---> __LOW LEVEL MODULES__).
      * First we create the low level modules --> bluetooth communication, XML parser, disk access, database connection.
      * Second ---> we implement the high level modules that rely heavily on the low level ones.
    * THIS IS THE INTUITION ... BUT ITS NOT GOOD.
    * What if we want to replace a low level module? We have to change the whole high level implementation as well.
      * For example: connect to Oracle instead of MySQL OR want to store data in XML instead of CSV.
    * Conclusion: high level modules should not depend on low level modules. We have to use abstraction and an abstract layer between the low and high level modules (__HIGH LEVEL MODULES <---> ABSTRACT LAYER (INTERFACES) <---> LOW LEVEL MODULES__).
  + Very difficult to apply: but it is the most important principle.
  + High level modules should not depend on low level modules... Both should depend on abstraction !!! Abstraction should not depend on detail... Details should depend upon abstraction !!!
    * ```java
      Dog dog = new Dog();
      // Create Animal interface
      Animal dog = new Dog();
      ```

## 2. Behavioral Design Patterns
- ###### <ins>_Strategy Pattern_</ins> ######
  + Very important principle #1 in design -> <ins>take what varies and encapsulate it ... and it will not affect the rest of our code.</ins>
  + The code will be much more flexible ... later you can alter or extend the parts that vary without affecting those that does not !!!
    * __Code that changes    <------->  code that stays the same__
  + Very important principle #2 in design -> <ins>program to an interface/supertype not an implementation!!! Abstract superclass would be perfect too...</ins>
    * The actual runtime object is not locked into the code.
    * The type of variable should be a supertype/interface: can be of any concrete implementation.
      * ```java
        Dog dog = new Dog();    // Not so good "programming to an implementation"
        Animal dog = new Dog();   // GOOD "programming to an interface"
        ```
    * Sometimes it's good to separate behaviors from implementation: easier to reuse.
    * We can add new behavior without modifying any of existing behavior classes!!!
  + Very important principle #3 in design -> <ins>Favor composition over inheritance</ins>!!!
    * Composition: HAS-A relationship -> it gives you a lot more flexibility, so composition is used in several design patterns.
      1. you can encapsulate stuffs into their own set of classes.
      2. YOU CAN CHANGE BEHAVIOR AT RUNTIME with interfaces.
    * Inheritance: IS-A relationship
- ###### <ins>_Observer Pattern_</ins> ######
  + Defines a one-to-many dependency -> if one object changes state all of its dependents are notified automatically.
    * The observers rely/dependent on the subjects.
  + Why is it good? LOOSE COUPLING !!!
    * When two objects are loosely coupled, they can interact but they have little knowledge of each other.
    * The only thing the subject knows about an observer is that it implements a certain interface.
    * We can add observers whenever we want: just have to implement the Observer interface.
    * We do not have to modify the subject to add new type of observers.
    * We can independently reuse subjects or observers.
    * We can change the subject or observer independently.
    * We can build flexible systems that can handle change because the interdependency between objects are minimal.
  + Very important principle in design -> __USE LOOSELY COUPLED DESIGNS BETWEEN OBJECTS THAT INTERACTS__
