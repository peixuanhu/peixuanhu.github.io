---
title: Effective Java-32. Use bounded wildcards to increase API flexibility
categories:
  - [Java, Effective Java]
---

# Item 32: Combine generics and varargs judiciously

Recall from Item 28 that a non-reifiable type is one whose runtime representa- tion has less information than its compile-time representation, and that **nearly all generic and parameterized types are non-reifiable**. 

If **a method declares its varargs parameter to be of a non-reifiable type**, the compiler generates a warning on the declaration. If **the method is invoked on varargs parameters whose inferred type is non-reifiable**, the compiler generates a warning on the invocation too. The warnings look something like this:

```java
warning: [unchecked] Possible heap pollution from
       parameterized vararg type List<String>
```

> *Heap pollution* occurs when a variable of a parameterized type refers to an object that is not of that type. It can cause the compiler’s automatically generated casts to fail, violating the fundamental guarantee of the generic type system.

Example:

```java
// Mixing generics and varargs can violate type safety! 
static void dangerous(List<String>... stringLists) {
  List<Integer> intList = List.of(42);
  Object[] objects = stringLists;
  objects[0] = intList; // Heap pollution 
  String s = stringLists[0].get(0); // ClassCastException
}
```

**It is unsafe to store a value in a generic varargs array parameter.**