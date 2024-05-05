---
title: Effective Java-14. Consider implmenting Comparable
categories:
  - [Java, Effective Java]
---

# Item 14: Consider implementing *Comparable*

If you are writing **a value class with an obvious natural ordering**, such as alphabetical order, numerical order, or chronological order, you should implement the `Comparable` interface:

```java
public interface Comparable<T> {
  int compareTo(T t);
}
```

## General contract of *compareTo* method

Compares this object with the specified object for order.

Returns a **negative integer, zero, or a positive integer** as this object is **less than, equal to, or greater than** the specified object.

Throws `ClassCastException` if the specified object’s type prevents it from being compared to this object.

- The implementor must ensure that `sgn(x.compareTo(y)) == -sgn(y. compareTo(x))` for all x and y.

- The implementor must also ensure that the relation is transitive: `(x. compareTo(y) > 0 && y.compareTo(z) > 0)` implies `x.compareTo(z) > 0`.

- Finally, the implementor must ensure that `x.compareTo(y) == 0` implies that `sgn(x.compareTo(z)) == sgn(y.compareTo(z))`, for all z.

- It is strongly recommended, but not required, that `(x.compareTo(y) == 0) == (x.equals(y))`.

  Generally speaking, any class that implements the `Comparable` interface and violates this condition should **clearly indicate this fact**. The recommended language is “Note: This class has a natural ordering that is inconsistent with equals.”

## *compareTo* and *equals*

Unlike the `equals` method, which imposes a global equivalence relation on all objects, `compareTo` **doesn’t have to work across objects of different types**: when confronted with objects of different types, `compareTo` is permitted to throw `ClassCastException`.

Writing a `compareTo` method is similar to writing an `equals` method, but there are a few key differences. Because the `Comparable` interface is parameterized, the `compareTo` method is statically typed, so you **don’t need to type check or cast its argument**. If the argument is of the wrong type, the invocation won’t even compile. If the argument is null, the invocation should throw a `NullPointerException`, and it will, as soon as the method attempts to access its members.

## Note

If a class has **multiple significant fields**, the order in which you compare them is critical. Start with **the most significant field** and work your way down.

```java
// Multiple-field Comparable with primitive fields
public int compareTo(PhoneNumber pn) {
   int result = Short.compare(areaCode, pn.areaCode);
   if (result == 0)  {
       result = Short.compare(prefix, pn.prefix);
       if (result == 0)
           result = Short.compare(lineNum, pn.lineNum);
	}
   return result;
}
```

In Java 8, the `Comparator` interface was outfitted with a set of *comparator construction methods*, which enable fluent construction of comparators. These comparators can then be used to implement a `compareTo` method, as required by the `Comparable` interface.

```java
// Comparable with comparator construction methods
private static final Comparator<PhoneNumber> COMPARATOR =
       comparingInt((PhoneNumber pn) -> pn.areaCode)
         .thenComparingInt(pn -> pn.prefix)
         .thenComparingInt(pn -> pn.lineNum);

public int compareTo(PhoneNumber pn) {
   return COMPARATOR.compare(this, pn);
}
```

> There are also comparator construction methods for object reference types. The static method, named `comparing`.

## *Comparator*

If a class does not implement `Comparable` or you need a nonstandard ordering, use a `Comparator` instead.

Example:

```java
// Comparator based on static compare method
 static Comparator<Object> hashCodeOrder = new Comparator<>() {
   public int compare(Object o1, Object o2) {
     return Integer.compare(o1.hashCode(), o2.hashCode());
   }
};
```

## Summary

In summary, whenever you implement a value class that has a sensible ordering, you should have the class implement the `Comparable` interface so that **its instances can be easily sorted, searched, and used in comparison-based collections.** 

When comparing field values in the implementations of the `compareTo` methods, avoid the use of the < and > operators. Instead, use the **static `compare` methods** in the boxed primitive classes or the **comparator construction methods** in the `Comparator` interface.

