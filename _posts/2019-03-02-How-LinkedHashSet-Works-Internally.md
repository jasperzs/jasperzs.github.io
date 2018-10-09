---
layout:     post
title:      How LinkedHashSet Works Internally
subtitle:   Understanding LinkedHashSet
date:       2019-03-02
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
    - Java
    - Data Structure
---

## 1. Overview
LinkedHashSet is an extended version of Hashset.

HashSet:
- doesn't follow any order
- uses HashMap internally to store elements

LinkedHashSet:
- maintains insertion order
- uses LinkedHashMap internally to store elements

## 2. Constructors
There are 4 constructors in LinkedHashSet class which are all simply calling to the same super class, HashSet, constructor.
```java
//Constructor - 1

public LinkedHashSet(int initialCapacity, float loadFactor)
{
      super(initialCapacity, loadFactor, true);              //Calling super class constructor
}

//Constructor - 2

public LinkedHashSet(int initialCapacity)
{
        super(initialCapacity, .75f, true);             //Calling super class constructor
}

//Constructor - 3

public LinkedHashSet()
{
        super(16, .75f, true);                //Calling super class constructor
}

//Constructor - 4

public LinkedHashSet(Collection<? extends E> c)
{
        super(Math.max(2*c.size(), 11), .75f, true);          //Calling super class constructor
        addAll(c);
}
```

This constructor in HashSet class is a package private constructor which is used only by LinkedHashSet. It takes initial capacity, load factor and one boolean dummy value as it’s arguments. This __boolean dummy value__ is just used to differentiate this constructor from other constructors of HashSet class which take initial capacity and load factor as their arguments.
```java
HashSet(int initialCapacity, float loadFactor, boolean dummy)
{
        map = new LinkedHashMap<>(initialCapacity, loadFactor);
}
```

As can be seen, this constructor internally creates one new LinkedHashMap object. This LinkedHashMap object is used by the LinkedHashSet to store it’s elements.

LinkedHashSet doesn’t have it’s own methods. All methods are inherited from it’s super class i.e HashSet. So. all operations on LinkedHashSet work in the same manner as that of HashSet. The only change is the internal object used to store the elements. In hashSet, elements you insert are stored as __keys of HashMap object__. Where as in LinkedHashSet, elements you insert are stored as __keys of LinkedHashMap object__. The values of these keys will be the same constant i.e __[“PRESENT“](https://javaconceptoftheday.com/how-hashset-works-internally-in-java/)__.

## 3. How LinkedHashSet Maintains Insertion Order?
LinkedHashSet uses LinkedHashMap object to store it’s elements. The elements you insert in the LinkedHashSet are stored as keys of this LinkedHashMap object. Each __key, value pair__ in the LinkedHashMap are instances of it’s static inner class called __Entry<K, V>__. This Entry<K, V> class extends __HashMap.Entry__ class. The insertion order of elements into LinkedHashMap are maintained by adding two new fields to this class. They are __before__ and __after__. These two fields hold the references to previous and next elements. These two fields make LinkedHashMap to function as a doubly linked list.
```java
private static class Entry<K,V> extends HashMap.Entry<K,V>
{
        // These fields comprise the doubly linked list used for iteration.
        Entry<K,V> before, after;

        Entry(int hash, K key, V value, HashMap.Entry<K,V> next) {
            super(hash, key, value, next);
        }
}
```

The first two fields of above inner class of LinkedHashMap – __before__ and __after__ are responsible for maintaining the insertion order of the LinkedHashSet. The head and tail fields of LinkedHashMap stores the head and tail of this doubly linked list, respectively.
```java
/**
* The head (eldest) of the doubly linked list.
*/
transient LinkedHashMap.Entry<K,V> head;

/**
* The tail (youngest) of the doubly linked list.
*/
transient LinkedHashMap.Entry<K,V> tail;
```

In LinkedHashMap, the same set of Entry objects (rather references to Entry objects) are arranged in two different manner. One is the HashMap and another one is Doubly linked list. The Entry objects just sit on heap memory, unaware of that they are part of two different data structures.

## 4. An Example
```java
public class LinkedHashSetExample
{
    public static void main(String[] args)
    {
        //Creating LinkedHashSet

        LinkedHashSet<String> set = new LinkedHashSet<String>();

        //Adding elements to LinkedHashSet

        set.add("BLUE");

        set.add("RED");

        set.add("GREEN");

        set.add("BLACK");
    }
}
```

![How LinkedHashSet Works Internally]({{ site.url }}/img/java/howLinkedHashSetWorksInternally/howLinkedHashSetWorksInternally.png)
