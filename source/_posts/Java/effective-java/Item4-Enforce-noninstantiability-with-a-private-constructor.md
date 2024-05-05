---
title: Effective Java-4. Enforce noninstantiability with a private constructor
categories:
  - [Java, Effective Java]
---

# Item 4:  Enforce noninstantiability with a private constructor

## Background

Write a class that is just a grouping of static methods and static fields.

Such *utility classes* were not designed to be instantiated: an instance would be nonsensical. In the absence of explicit constructors, however, the compiler pro- vides a public, parameterless *default constructor*. To a user, this constructor is indistinguishable from any other. It is not uncommon to see unintentionally instantiable classes in published APIs.

## Approaches for enforcing noninstantiability

### 1. Making a class abstract

**Attempting to enforce noninstantiability by making a class abstract does not work.** 

The class can be subclassed and the subclass instantiated. Furthermore, it misleads the user into thinking the class was designed for inheritance (Item 19).

### 2. Including a private constructor

```java
// Noninstantiable utility class
public class UtilityClass {
  // Suppress default constructor for noninstantiablity
  private UtilityClass() {
    throw new AssertionError();
  }
  ... // Remainder omitted
}
```

#### Explanation

Because the explicit constructor is **private**, it is **inaccessible outside the class.**

The **AssertionError** isn’t strictly required, but it provides insurance in case the constructor is accidentally invoked from within the class. It guarantees the class will never be instantiated under any circumstances.

This idiom is mildly counter- intuitive because the constructor is provided expressly so that it cannot be invoked. It is therefore wise to include a **comment**.

#### Disdvantages

This idiom also prevents the class from being **subclassed**. All constructors must invoke a superclass constructor, explicitly or implicitly, and a subclass would have no accessible superclass constructor to invoke.