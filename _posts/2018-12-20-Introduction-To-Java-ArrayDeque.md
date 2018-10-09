---
layout:     post
title:      Introduction to Java ArrayDeque
subtitle:   Understanding Java ArrayDeque data structure
date:       2018-12-20
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
    - Java
    - Data Structure
---

## 1. Overview

An _ArrayDeque_ (also known as an “Array Double Ended Queue”, pronounced as “ArrayDeck”) __is a special kind of a growable array that allows us to add or remove an element from both sides__.

An _ArrayDeque_ implementation can be used as a _Stack_ (Last-In-First-Out) or a _Queue_ (First-In-First-Out). Example can be found [here](https://github.com/eugenp/tutorials/tree/master/core-java-collections).


## 2. The API at a Glance

For each operation, we basically have two options.

The first group consists of methods that throw exception if the operation fails. The other group returns a status or a value:

| Operation | Method | Method Throwing Exception |
| :--- | :--- | :--- |
| Insertion from Head | _offerFirst(e)_ | _addFirst(e)_ |
| Removal from Head | _pollFirst()_ | _removeFirst()_ |
| Retrieval from Head | _peekFirst_ | _getFirst()_ |
| Insertion from Tail | _offerLast(e)_ | _addLast(e)_ |
| Removal from Tail | _pollLast()_ | _removeLast()_ |
| Retrieval from Tail | _peekLast()_ | _getLast()_ |

## 3. Using Methods

### 3.1 Using ArrayDeque as a Stack

Push an element:
```java
@Test
public void whenPush_addsAtFirst() {
    Deque<String> stack = new ArrayDeque<>();
    stack.push("first");
    stack.push("second");

    assertEquals("second", stack.getFirst());
}
```

Pop an element:
```java
@Test
public void whenPop_removesLast() {
    Deque<String> stack = new ArrayDeque<>();
    stack.push("first");
    stack.push("second");

    assertEquals("second", stack.pop());
}
```

The _pop_ method throws NoSuchElementException when a stack is empty.

### 3.2 Using ArrayDeque as a Queue

Offer an element:
```java
@Test
public void whenOffer_addsAtLast() {
    Deque<String> queue = new ArrayDeque<>();
    queue.offer("first");
    queue.offer("second");

    assertEquals("second", queue.getLast());
}
```

Poll an element:
```java
@Test
public void whenPoll_removesFirst() {
    Deque<String> queue = new ArrayDeque<>();
    queue.offer("first");
    queue.offer("second");

    assertEquals("first", queue.poll());
}
```

The _poll_ method returns a _null_ value if a queue is empty.


## 4. How's ArrayDeque Implemented

![ArrayDeque Overview]({{ site.url }}/img/java/arrayDeque/overview.jpg)

Under the hood, the __*ArrayDeque* is backed by an array which doubles its size when it gets filled__.

Initially, the array is initialized with a size of 16. It’s implemented as a double-ended queue where it maintains two pointers namely a head and a tail.

### 4.1 ArrayDeque as Stack

![ArrayDeque as Stack]({{ site.url }}/img/java/arrayDeque/arrayDequeAsStack.jpg)

As can be seen, when a user adds in an element using the _push_ method, it moves the head pointer by one.

When we pop an element, it sets the element at the head position as _null_ so the element could be garbage collected, and then moves back the head pointer by one.


### 4.2 ArrayDeque as Queue

![ArrayDeque as Queue]({{ site.url }}/img/java/arrayDeque/arrayDequeAsQueue.jpg)

When we add in an element using the _offer_ method, it moves the tail pointer by one.

While when user polls an element, it sets the element at the head position to null so the element could be garbage collected, and then moves the head pointer.


### 4.3 Notes on ArrayDeque

Finally, a few more notes worth understanding and remembering about this particular implementation:
- It’s not thread-safe
- Null elements are not accepted
- Works significantly faster than the synchronized Stack
- Is a faster queue than LinkedList due to the better locality of reference
- Most operations have amortized constant time complexity
- An Iterator returned by an ArrayDeque is fail-fast
- ArrayDeque automatically doubles the size of an array when head and tail pointer meets each other while adding an element
