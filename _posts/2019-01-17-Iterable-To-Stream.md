---
layout:     post
title:      Iterable to Stream in Java
subtitle:   Example of StreamSupport
date:       2019-01-17
author:     Mr.Humorous 🥘
header-img: img/java/header.jpg
catalog: true
tags:
    - Java
---

## 1. Converting Iterable to Stream
The `Iterable` interface is designed keeping generality in mind and does not provide any `stream()` method on its own.

Simply put, you can pass it to `StreamSupport.stream()` method and get a `Stream` from the given `Iterable instance`.

Let’s consider our `Iterable` instance:
```java
Iterable<String> iterable
  = Arrays.asList("Testing", "Iterable", "conversion", "to", "Stream");
```

And here’s how we can convert this `Iterable` instance into a Stream:
```java
StreamSupport.stream(iterable.spliterator(), false);
```

Note that the second param in `StreamSupport.stream()` determines if the resulting `Stream` should be parallel or sequential. You should set it true, for a parallel `Stream`.

Now, let's test our implementation:
```java
@Test
public void givenIterable_whenConvertedToStream_thenNotNull() {
    Iterable<String> iterable
      = Arrays.asList("Testing", "Iterable", "conversion", "to", "Stream");

    Assert.assertNotNull(StreamSupport.stream(iterable.spliterator(), false));
}
```

Also, a quick side-note – streams are not reusable, while `Iterable` is; it also provides a `spliterator()` method, which returns a `java.lang.Spliterator` instance over the elements described by the given `Iterable`.

## 2. Performing Stream Operation
```java
@Test
public void whenConvertedToList_thenCorrect() {
    Iterable<String> iterable
      = Arrays.asList("Testing", "Iterable", "conversion", "to", "Stream");

    List<String> result = StreamSupport.stream(iterable.spliterator(), false)
      .map(String::toUpperCase)
      .collect(Collectors.toList());

    assertThat(
      result, contains("TESTING", "ITERABLE", "CONVERSION", "TO", "STREAM"));
}
```
