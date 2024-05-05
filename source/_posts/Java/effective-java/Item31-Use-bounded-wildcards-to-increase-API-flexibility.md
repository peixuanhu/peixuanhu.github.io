---
title: Effective Java-31. Use bounded wildcards to increase API flexibility
categories:
  - [Java, Effective Java]
---

# Item 31: Use bounded wildcards to increase API flexibility

## Get and Put Principle

**For maximum flexibility, use wildcard types on input parameters that represent producers or consumers.**

**PECS** stands for **producer-extends, consumer-super**.

> In other words, if a parameterized type represents a T producer, use `<? extends T>`; if it represents a T consumer, use `<? super T>`.

Example:

```java
public static <T extends Comparable<? super T>> T max( List<? extends T> list)
```



## Note

1. **Do not use bounded wildcard types as return types.** 

   Example:

   ````java
    public static <E> Set<E> union(Set<? extends E> s1, Set<? extends E> s2)
   ````

   The return type is `Set<E>`.

2. **If a type parameter appears only once in a method declaration, replace it with a wildcard.**

   If it’s an unbounded type parameter, replace it with an unbounded wildcard; 

   if it’s a bounded type parameter, replace it with a bounded wildcard.

   Example:

   ```java
   // Two possible declarations for the swap method
   public static <E> void swap(List<E> list, int i, int j); 
   public static void swap(List<?> list, int i, int j);
   ```

   > In a public API, the second is better because it’s simpler.



## Summary

In summary, using wildcard types in your APIs, while tricky, makes the APIs far more flexible. 

> If you write a library that will be widely used, the proper use of wildcard types should be considered mandatory. 

Remember the basic rule: **producer-extends, consumer-super (PECS)**. 

Also remember **that all comparables and comparators are consumers**.