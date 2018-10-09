---
layout:     post
title:      Introduction to Interface
subtitle:   Java 8 Interfaces
date:       2019-06-02
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
    - Java
---

In an interface all the fields (variables) are by default _public_, _static_ and _final_. For an example in the following code both the MIN_SIZE and MAX_SIZE are _public_, _static_ and _final_ constants.
```java
interface Size {
    int MIN_SIZE = 1;
    public static final int MAX_SIZE = 10;
}
```

Up to Java 1.7 version, all the methods declared in interfaces are _public_ and _abstract_ by default. Since Java 1.8, an interface can have _default_ methods and _static_ methods as well. Therefore, the updated rule is:

> An interface can have default methods and static methods. Any other methods are implicitly public and abstract. All the fields declared in an interface are implicitly public, static and final constants.

```java
interface Super {
    /**
     * An abstract method. By default it is public and abstract.
     */
    void print();

    /**
     * Default method, introduced in Java 8.
     */
    public default void doStuff() {
        System.out.println("Hello world");
    }

    /**
     * Static method in interface, introduced in Java 8.
     */
    public static void sayHello() {
        System.out.println("Hello");
    }
}
```

In the above code a sub class object is needed to access the abstract method _print_ and the default method _doStuff_. The static method _sayHello_ can be called directly using the interface name without any sub classes. Sub classes must override the abstract methods and optionally they can override the default methods as shown below. In the following code, sub class _Base_ overrides the abstract method only.
```java
class Base implements Super {
    /**
     * Override the abstract method.
     */
    @Override
    public void print() {
        System.out.println("Base");
    }
}
```

Interfaces are used to provide a template which must be implemented by other entities later. A best real world example for interface is USB port. All the computers have USB ports, and by having a USB port they assure that any device which has a USB pin can connect to this port. Computer knows how to transfer data through the USB port but it never cares about the actual device. The implementation depends on the device which is connected to the port. In Java it can be simulated using interface as given below.
```java
public class Computer {
    public static void main(String[] args) {
        Computer comp = new Computer();
    }

    public void connect(USB usb) {
        usb.send(new byte[] {1});   // Send some data
        byte[] data = usb.receive();    // Receive some data
        // do something here
    }
}

interface USB {
    void send(byte[] data);

    byte[] receive();
}
```

Any class which wants to connect to this computer can simply implement the USB interface and provide the necessary implementation as shown below.
```java
class Mouse implements USB {
    @Override
    public void send(byte[] data) {
        System.out.println("Connected");
    }

    @Override
    public byte[] receive() {
        return new byte[] {120, 87};
    }
}

public class Computer {
    public static void main(String[] args) {
        Computer comp = new Computer();
        Mouse mouse = new Mouse();
        comp.connect(mouse);
    }

    public void connect(USB usb) {
        usb.send(new byte[] {1});   // Send some data
        byte[] data = usb.receive();    // Receive some data
        // do something here
    }
}

interface USB {
    void send(byte[] data);

    byte[] receive();
}
```

To have a better understanding, consider another problem. A _DownloadManager_ library wants to pass the downloaded percentage to some other entities. Based on the requirements, the developers who use this library may display the downloaded percentage in a GUI progress bar or in a command window. In this case the _DownloadManager_ library developer cannot determine the implementation of other developers, who use this library. Now the problem is: how to establish an agreement to pass the downloaded percentage to anyone interested? Following code is a model (Do not try to compile this code) which simulates this problem and solves it using an interface _OnDownloadListener_ with a single method _onDownload_.
```java
public class DownloadManager {
    private OnDownloadListener listener;

    public void setListener(OnDownloadListener listener) {
        this.listener = listener;
    }

    public void start() {
        boolean completed = false;
        double percent = 0.0;
        while (!completed) {
            // download the file
            // increase the completed percentage
            if (listener != null) {
                listener.onDownload(percent);
            }
        }
    }
}

interface OnDownloadListener {
    void onDownload(double percent);
}
```

By providing an interface, the developer knows that all of its sub-classes have an implementation of _onDownload_ method. Whenever the _onDownload_ method is called with the current percentage value, other developers' own implementation will be executed. In this case, the _DownloadManager_ library developer can develop his/her project without caring about the actual implementation which is going to be bound at the runtime.

There is one more advantage of having interfaces in Java. Object Oriented Programming supports multiple inheritance; in other words a sub class can have more than one super class. In Java multiple class to class inheritance is prohibited to avoid dead diamond problem and Interface is used to implement the multiple inheritance. Look at the code which simulates a sample problem given in [Wikipedia](http://en.wikipedia.org/wiki/Virtual_inheritance).
```java
interface Animal {
    void eat();
}

interface Mammal extends Animal {
    void breastFeed();
}

interface WingedAnimal extends Animal {
    void flap();
}

/**
 * Multiple inheritance:
 * Bat implements both Mammal and WingedAnimal.
 */
class Bat implements Mammal, WingedAnimal {
    @Override
    public void eat() {
        System.out.println("Eating");
    }

    @Override
    public void breastFeed() {
        System.out.println("Feeding");
    }

    @Override
    public void flap() {
        System.out.println("Flying");
    }
}
```

In this example _Bat_ implements both _Mammal_ and _WingedAnimal_ interfaces. Therefore Bat can be sent to a method with a parameter _Mammal_ and to another method with a parameter _WingedAnimal_ parameter. In other words Bat IS-A Mammal as well as Bat IS-A WingedAnimal.
```java
public class MultipleInheritance {
    public static void main(String[] args) {
        Bat bat = new Bat();
        doFly(bat);
        doFeed(bat);
    }

    public static void doFly(WingedAnimal ani) {
        ani.flap();
    }

    public static void doFeed(Mammal ani) {
        ani.breastFeed();
    }
}
```

### Java 8 Features
From Java 8, Interface supports default and static methods. Default methods are already covered in this [article](http://www.javahelps.com/2015/01/default-methods.html), so the static methods are highly focused in this tutorial. In Java utility classes must define all their methods as static methods and the constructor must be private to avoid instantiation of that class.  For example java.util.Arrays is an utility class where all the methods are static and the constructor is private. Utility class developers take some extra effort to avoid the instantiation of the utility class. From Java 8 there are no need for a utility class with private constructor; it can be simply replaced by an interface with static methods. Look at the example where a utility interface is provided to print any String arrays in a formatted manner.
```java
public class PrintArray {
    public static void main(String[] args) {
        String[] arr = {"Java", "C++", "C", "Python", "PHP"};
        Printer.print(arr);
    }
}

interface Printer {
    public static void print(String[] array) {
        StringJoiner joiner = new StringJoiner(", ", "[", "]");
        for (String str : array) {
            joiner.add(str);
        }
        System.out.println(joiner.toString());
    }
}
```

#### Do and Don't
Whenever you need to provide a template, use the interface. Coding for interface is one of the best practice, so try to have an interface as the top most element in a class hierarchy. However you must be aware of what you are doing. Since Java 8 allows to have _default_ and _static_ methods in an interface, there is a high risk of polluting an interface by using default and static methods without any valid reasons. Always remember these two rules when you are going to use default or static methods in an interface.

##### Rule #1:
Do not use static methods in an interface until that method is a utility method.

##### Rule #2:
Do not use default methods until you really need to provide a default implementation for a method. Default methods are commonly useful in extending the functionality of an existing interface. Do not use default methods instead of ordinary instance methods because interfaces with default methods are open for dead diamond problem. If you really need to use both abstract and non-abstract instance methods use the abstract class not the interface.
