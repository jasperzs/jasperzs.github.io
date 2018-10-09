---
layout:     post
title:      Java Equivalents of Malloc(), new, free(), and delete
subtitle:   Memory management operators in Java
date:       2018-12-10
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
    - Java
---

## 1. The <ins>_delete_</ins> operator in Java...?

__Java has no direct equivalent of delete.__ That is, there is no call that you can make to tell the JVM to "deallocate this object". It is up to the JVM to work out when an object is no longer referenced, and then deallocate the memory at its convenience.

Now if you're a C++ programmer, at this point you might be thinking "that's cool, but what about deconstructors"? In C++, if a class has a deconstructor, it is called at the point of invoking delete and allows any cleanup operations to be carried out. In Java, things are slightly different:

- there's __no way to guarantee__ that any cleanup function will be called at the point of deallocation...
- ...but given Java's automatic garbage collection, cleanup functions are generally __much less necessary__;
- Java provides a mechanism called __finalization__, which is essentially an "__emergency__" cleanup function with a __"weak" guarantee__ of being called;
- Java's _finally_ mechanism provides another means of performing certain common cleanup functions (such as closing files).

An explicit deconstructor isn't needed in Java to deallocate "subobjects". In Java, if object A references object B, but nothing else does, then object B will also be eligible for automatic garbage collection at the same time as A. So there's no need for a finalizer (nor would it be possible to write one) to explicitly deallocate object B.

## 2. Garbage collection and finalization

The end of an object's life cycle in Java generally looks as follows:
- at some point, the JVM determines that the object is no longer reachable: that is, there are no more references to it, or at least, no more references that will ever be accessed; at this point, the object is deemed __eligible for garbage collection__;
- if the object has a __finalizer__, then it is __scheduled for finalization__;
- if it has one, then at some later point in time, the __finalizer may be called__ by some arbitrary thread allocated by the JVM for this purpose;
- on its "next pass", the garbage collector will check that the finalized object is __still unreachable__;
- then at some __arbitrary point in time__ in the future— but always _after_ any finalization — the memory occupied by the object may actually deallocated.

You'll notice that there are a few "mays" and "arbitraries" in this description. In particular, there are a few implications that we need to be aware of if we use a finalizer:
- the __finalizer may never actually get called__, even if the object becomes unreachable;
- having a finalizer adds some extra steps to an object's life cycle so that in general, __finalizers delay object deallocation__;
- because the object needs to be accessed later, finalizers __prevent optimisations__ that can be made to allocation/deallocation: for example, our notion that the JVM can put an object on the stack goes out the window if the object will later need to be accessed by a separate finalizer thread at some arbitrary point in time.

So finalizers should really only be considered an __"emergency" cleanup__ operation used on exceptional objects. A typical example is something like __file stream__ or __network socket__. Although they provide a `close()` method, and we should generally try to use it, in the worst case, it's better than nothing to have the stream closed at some future point when the stream object goes out of scope. And we know that in the very worst case, the stream or socket will be closed when our application exits— at least on sensible operating systems. This last point is important: the finalizer may not get called at all even in "normal" operation, so it's really no good putting an operation in a finalizer whose execution is crucial to the system behaving once the application exits.

Another typical use of a finalizer is for objects linked to some __native code__ that __allocates resources outside the JVM__. The JVM can't automatically clean up or garbage collect such resources.

## 3. How to create a finalizer

So despite all the warnings, if you still want to create a finalizer, then essantially you need to __override the finalize() method__ of the given class. (Strictly speaking, every class otherwise inherits an empty `finalize()` method from `Object`, but good VMs can "optimise away" the default empty finalizer.) A typical finalizer looks as follows:
```java
protected void finalize() throws Throwable {
  super.finalize();
  ... cleanup code, e.g. deleting a temporary file,
      calling a native deallocate() method etc ...
}
```

General points about coding finalizers:
- you should always call the __superclass finalize()__ in case it needs to do some aditional cleanup;
- you should always __synchronize on shared resources__, as finalizers can be run concurrently;
- you should make __no expectations about ordering__ of finalizers: if two objects are garbage collectable, there is no fixed order in which their finalizers will be run (and they could be run concurrently);
- finalizers should __run quickly__, as they could otherwise delay the finalization (and hence garbage collection) of other objects;
- you should __not rely on seeing any exception__ thrown by a finalizer — the JVM will generally swallow them (but they will still interrupt the finalization of the object in question, of course).

## 4. Alternatives to finalizers

In general, it's better to use an alternative to `finalize()` methods where possible— they really are a last resort. Safer alternatives include:

- for objects with a well-defined lifetime, use an __explicit close or cleanup method__;
- where useful, call the cleanup method in a finally block;
- using a __shutdown hook__ (see Runtime.addShutdownHook()) to perform actions such as deleting temporary files that just need to be done "when the application exits".

And of course, don't expect miracles. If the VM segfaults or there's a power cut, there's no cleanup method or shutdown hook that's going to help you. If you are writing some critical application that requires "no data loss", you will have to take other steps (such as using a before-and-after transaction log) to minimise potential problems caused by abnormal shutdown.
