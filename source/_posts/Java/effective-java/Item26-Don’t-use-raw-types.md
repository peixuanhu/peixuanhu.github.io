---
title: Effective Java-26. Don’t use raw types
categories:
  - [Java, Effective Java]
---

# Item 26: Don’t use raw types

It is legal to use raw types (generic types without their type parameters), but you should never do it. **If you use raw types, you lose all the safety and expressiveness benefits of generics.**

> For compatibility, Java designers permit raw types in the first place.

## Terms

- A class or interface whose declaration has one or more *type parameters* is a ***generic* class** or **interface**.

  > For example, the `List` interface has a single type parameter, E, representing its element type. The full name of the interface is `List<E>` (read “list of E”), but people often call it `List` for short.

- Generic classes and interfaces are collectively known as ***generic types***.

- Each generic type defines a set of ***parameterized types***, which consist of the class or interface name followed by an angle-bracketed list of *actual type parameters* corresponding to the generic type’s formal type parameters.

  > For example, `List<String>` (read “list of string”) is a parameterized type representing a list whose elements are of type String. (String is the actual type parameter corresponding to the formal type parameter E.)

- Each generic type defines a ***raw type***, which is the name of the generic type used without any accompanying type parameters.

  > They exist primarily for compatibility with pre-generics code.

## Raw types vs *Object* parameterized types

**You lose type safety if you use a raw type such as `List`, but not if you use a parameterized type such as `List<Object>`.**

While you shouldn’t use raw types such as `List`, it is fine to use types that are parameterized to allow insertion of arbitrary objects, such as `List<Object>`. What is the difference between the raw type `List` and the parameterized type `List<Object>`? Loosely speaking, `List` has **opted out of the generic type system**, while `List<Object>` has explicitly told the compiler that it is capable of **holding objects of any type**.

While you can pass a `List<String>` to a parameter of type `List`, you can’t pass it to a parameter of type `List<Object>`. There are **subtyping rules** for generics, and **`List<String>` is a subtype of the raw type `List`, but not of the parameterized type `List<Object>`** (Item 28).

Example:

```java
public static void main(String[] args) {
  List<String> strings = new ArrayList<>(); unsafeAdd(strings, Integer.valueOf(42));
  String s = strings.get(0);
}

/*
Test.java:10: warning: [unchecked] unchecked call to add(E) as a member of the raw type List
       list.add(o);
               ^
*/
private static void unsafeAdd1(List list, Object o) { 
  list.add(o);
}

/*
Test.java:5: error: incompatible types: List<String> cannot be converted to List<Object>
   unsafeAdd(strings, Integer.valueOf(42));
       ^
*/
private static void unsafeAdd2(List<Object> list, Object o) { 
  list.add(o);
}
```

If you run `unsafeAdd1` version of the program, you get a `ClassCastException` when the program tries to cast the result of the invocation `strings.get(0)`, which is an Integer, to a String.

If you run `unsafeAdd2` version of the program, and try to recompile the program, you’ll find that it no longer compiles but emits an *incompatible types* error message.

## Unbounded wildcard types <?>

If you want to use a generic type but you don’t know or care what the actual type parameter is, you can use a **question mark** instead.

> For example, the unbounded wildcard type for the generic type `Set<E>` is `Set<?>` (read “set of some type”). It is the most general parameterized `Set` type, capable of holding *any* set.

> `List<Integer>` and `List<String>` are not subtypes of `List<Object>`. So use `List<?>` as an alternative.

Example:

```java
// Use of raw type for unknown element type - don't do this! 
static int numElementsInCommon(Set s1, Set s2) {
   int result = 0;
   for (Object o1 : s1)
       if (s2.contains(o1))
           result++;
   return result;
}

// Uses unbounded wildcard type - typesafe and flexible 
static int numElementsInCommon(Set<?> s1, Set<?> s2) { ... }
```

### Note

**You can’t put any element (other than null) into a `Collection<?>`.**

## Exceptions to not using raw types

1. **You must use raw types in class literals.** The specification does not permit the use of parameterized types (though it does permit array types and primitive types). 

   In other words, `List.class`, `String[].class`, and `int.class` are all legal, but `List<String>.class` and `List<?>.class` are not.

2. **Use the `instanceof` operator on raw types.** Because generic type information is erased at runtime, it is illegal to use the instanceof operator on parameterized types.

   > Unbounded wildcard types is permitted when using `instanceof` operator, but in this case, the angle brackets and question marks are just noise.

   Example:

   ```java
   // Ligitimate use of raw type - instanceof operator
   if (o instanceof Set) { // Raw type
     Set<?> s = (Set<?>) o; // Wildcard type
   }
   ```

   **This is the preferred way to use the** **instanceof** **operator with generic types:**

   Note that once you’ve determined that o is a Set, you must cast it to the wildcard type Set<?>, not the raw type Set. This is a checked cast, so it will not cause a compiler warning.

## Summary

In summary, using raw types can lead to exceptions at runtime, so don’t use them. They are provided only for compatibility and interoperability with legacy code that predates the introduction of generics.

As a quick review:

- `Set<Object>` is a parameterized type representing a set that can contain objects of **any type**; 
- `Set<?>` is a wildcard type representing a set that can contain only objects of some **unknown type**;
- `Set` is a raw type, which opts out of the generic type system. 

The first two are safe, and the last is not.