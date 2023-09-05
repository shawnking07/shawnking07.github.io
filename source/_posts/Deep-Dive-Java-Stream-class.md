---
title: 'Deep Dive: Java Stream class'
date: 2023-09-05 14:30:26
tags:
---

In the realm of functional programming, Stream has gained immense popularity, and Java 8 gave it to us. Even though it’s been with us for some years now, not everyone has mastered using it. Let’s take a deep dive into the Java Stream Class.

## The Stream Class

Introduced in Java 8, Stream is a sequence of elements supporting sequential and parallel aggregate operations. It helps in achieving functionality like SQL statements. However, contrary to popular belief, it’s not a data structure but more of a view onto a data structure.

```java
Stream<String> stream = Stream.of("Java", "Python", "C++", "JavaScript");
```

## Note

Stream operations exploit the capability of multi-core processors, where each task is assigned to a separate core of the processor.

### Stream Class Methods

Below are examples of some methods available in the Stream class:

1. `forEach()`

`forEach()` method helps in iterating over each element of the stream.

```java
stream.forEach(elem -> System.out.println(elem));
```

2. `map()`

`map()` method is used to convert - or map - a Stream of size ‘n’ to another Stream of size ‘n’.

```java
stream.map(String::toUpperCase).forEach(System.out::println);
```

3. `filter()`

`filter()` method is used for filtering data based on certain conditions.

```java
stream.filter(s -> s.startsWith("J")).forEach(System.out::println);
```

4. `sorted()`

`sorted()` method is used to sort a Stream.

```java
stream.sorted().forEach(System.out::println);
```

## Conclusion

The power of Stream lies in its ability to exploit the declarative programming approach, making the best use of multi-core architecture. It might seem a little complex initially, but with practice, you will realize its benefits. Keep practicing and improving, Happy Coding!

```java
Stream<String> stream = Stream.of("Java", "is", "Love");
stream.forEach(System.out::println);
```

Remember, `stream()` away until you master the flow.