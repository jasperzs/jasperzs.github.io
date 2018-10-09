---
layout:     post
title:      ArrayList vs. LinkedList vs. Vector
subtitle:   Differences between List implementations
date:       2019-02-28
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
    - Java
    - Data Structure
---

## 1. List Overview
List is an ordered sequence of elements. Whereas, Set is a set of elements which is unordered and every element is unique.

![Class Hierarchy Diagram of Collection]({{ site.url }}/img/java/arrayListLinkedListVector/classHierarchyDiagramOfCollection.jpeg)

## 2. ArrayList vs. LinkedList vs. Vector
__Vector:__
- similar as ArrayList, but is synchronized
- doubles array size when more space is required to add more elements
- better choice than ArrayList when program is non-thread-safe

__ArrayList:__
- resizable array. As more elements are added to ArrayList, its size is increased dynamically
- elements can be accessed directly by using the get and set methods as it is an array
- grow 50% of its size when more space is required to add more elements
- default initial capacity is pretty small, so it is a good habit to construct the ArrayList with a higher initial capacity to avoid the resizing cost
- better choice than Vector when program is thread-safe

__LinkedList:__
- double linked list
- Performance on add and remove is better than ArrayList, but worse on get and set methods
- implements Queue interface which adds more methods than ArrayList and Vector, such as offer(), peek(), poll(), etc

## 3. ArrayList Example
```java
ArrayList al = new ArrayList();
al.add(3);
al.add(2);
al.add(1);
al.add(4);
al.add(5);
al.add(6);
al.add(6);

Iterator iter1 = al.iterator();
  while(iter1.hasNext()){
  System.out.println(iter1.next());
}
```

## 4. LinkedList Example
```java
LinkedList ll = new LinkedList();
ll.add(3);
ll.add(2);
ll.add(1);
ll.add(4);
ll.add(5);
ll.add(6);
ll.add(6);

Iterator iter2 = al.iterator();
  while(iter2.hasNext()){
  System.out.println(iter2.next());
}
```

As shown in the examples above, they are similar to use. The real difference is their underlying implementation and their operation complexity.

## 5. Vector Example
Vector is almost identical to ArrayList, and the difference is that Vector is synchronized. Because of this, it has an overhead than ArrayList. Normally, most Java programmers use ArrayList instead of Vector because they can synchronize explicitly by themselves.

## 6. Performance of ArrayList vs. LinkedList
The time complexity comparison is as follows:
![Time Complexity of ArrayList vs. LinkedList]({{ site.url }}/img/java/arrayListLinkedListVector/arrayListLinkedListPerformance.png)

```java
ArrayList arrayList = new ArrayList();
LinkedList linkedList = new LinkedList();

// ArrayList add
long startTime = System.nanoTime();

for (int i = 0; i < 100000; i++) {
  arrayList.add(i);
}
long endTime = System.nanoTime();
long duration = endTime - startTime;
System.out.println("ArrayList add:  " + duration);

// LinkedList add
startTime = System.nanoTime();

for (int i = 0; i < 100000; i++) {
  linkedList.add(i);
}
endTime = System.nanoTime();
duration = endTime - startTime;
System.out.println("LinkedList add: " + duration);

// ArrayList get
startTime = System.nanoTime();

for (int i = 0; i < 10000; i++) {
  arrayList.get(i);
}
endTime = System.nanoTime();
duration = endTime - startTime;
System.out.println("ArrayList get:  " + duration);

// LinkedList get
startTime = System.nanoTime();

for (int i = 0; i < 10000; i++) {
  linkedList.get(i);
}
endTime = System.nanoTime();
duration = endTime - startTime;
System.out.println("LinkedList get: " + duration);


// ArrayList remove
startTime = System.nanoTime();

for (int i = 9999; i >=0; i--) {
  arrayList.remove(i);
}
endTime = System.nanoTime();
duration = endTime - startTime;
System.out.println("ArrayList remove:  " + duration);

// LinkedList remove
startTime = System.nanoTime();

for (int i = 9999; i >=0; i--) {
  linkedList.remove(i);
}
endTime = System.nanoTime();
duration = endTime - startTime;
System.out.println("LinkedList remove: " + duration);
```

And the output is:
```
ArrayList add: 13265642
LinkedList add: 9550057
ArrayList get: 1543352
LinkedList get: 85085551
ArrayList remove: 199961301
LinkedList remove: 85768810
```

LinkedList is faster in add and remove, but slower in get. In brief, LinkedList should be preferred if:
- there are no large number of random access of element
- there are a large number of add/remove operations
