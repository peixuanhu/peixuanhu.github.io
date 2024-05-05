---
title: Effective Java-6. Avoid creating unnecessary objects
categories:
  - [Java, Effective Java]
---

# Item 6: Avoid creating unnecessary objects

## Reuse object instead of creating new equivalent object

It is often appropriate to reuse a single object instead of creating a new functionally equivalent object each time it is needed. Reuse can be both faster and more stylish. An object can always be reused if it is **immutable** (Item 17). In addition, you can also reuse mutable objects if you know they won’t be modified.

### Example

For example, suppose you want to write a method to determine whether a string is a valid Roman numeral.

The easiest way:

```java
// Performance can be greatly improved
static boolean isRomanNumeral(String s) {
  return s.matches("^(?=.)M*(C[MD]|D?C{0,3})"
		+ "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
}
```

**While** **String.matches** **is the easiest way to check if a string matches a regular expression, it’s not suitable for repeated use in performance-critical situations.** The problem is that it internally creates a Pattern instance for the regular expression and uses it only once, after which it becomes eligible for garbage collection.

To improve the performance, explicitly compile the regular expression into a Pattern instance (which is immutable) as part of class initialization, cache it, and reuse the same instance for every invocation of the isRomanNumeral method:

```java
// Reusing expensive object for improved performance
public class RomanNumerals {
  private static final Pattern ROMAN = Pattern.compile(
           "^(?=.)M*(C[MD]|D?C{0,3})"
           + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");
  static boolean isRomanNumeral(String s) {
      return ROMAN.matcher(s).matches();
	}
}
```



## Use *static factory methods* in preference to *constructors* on immutable classes

You can often avoid creating unnecessary objects by using *static factory methods* (Item 1) in preference to constructors on **immutable classes** that provide both. The constructor *must* create a new object each time it’s called, while the factory method is never required to do so and won’t in practice.

> For example, the factory method Boolean.valueOf(String) is preferable to the constructor Boolean(String), which was deprecated in Java 9. 

## Prefer primitives to boxed primitives, and watch out for unintentional autoboxing

**Autoboxing blurs but does not erase the distinction between primitive and boxed primitive types.**

Consider the following method, which calculates the sum of all the positive int values. To do this, the program has to use long arithmetic because an int is not big enough to hold the sum of all the positive int values:

```java
// Hideously slow! Can you spot the object creation?
private static long sum() {
  Long sum = 0L;
  for (long i = 0; i <= Integer.MAX_VALUE; i++)
     sum += i;
  return sum;
}
```

The variable sum is declared as a Long instead of a long, which means that the program constructs about 231 unnecessary Long instances (roughly one for each time the long i is added to the Long sum).

The lesson is clear: **prefer primitives to boxed primitives, and watch out for unintentional autoboxing.**

## Additional information

This item should not be misconstrued to imply that object creation is expensive and should be avoided. On the contrary, the creation and reclamation of small objects whose constructors do little explicit work is cheap, especially on modern JVM implementations. **Creating additional objects to enhance the clarity, simplicity, or power of a program is generally a good thing.**

Conversely, avoiding object creation by maintaining your own *object pool* is a bad idea unless the objects in the pool are extremely heavyweight. The **classic example** of an object that *does* justify an object pool is a **database connection**. The cost of establishing the connection is sufficiently high that it makes sense to reuse these objects. 
