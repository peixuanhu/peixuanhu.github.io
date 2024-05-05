---
title: Effective Java-27. Eliminate unchecked warnings
categories:
  - [Java, Effective Java]
---

# Item 27: Eliminate unchecked warnings

When you program with generics, you will see many compiler warnings: unchecked cast warnings, unchecked method invocation warnings, unchecked parameterized vararg type warnings, and unchecked conversion warnings.

## Rule

**Eliminate every unchecked warning that you can.** If you eliminate all warnings, you are assured that your code is typesafe, which is a very good thing. It means that you won’t get a `ClassCastException` at runtime.

### Suppress a warning

**If you can’t eliminate a warning, but you can prove that the code that provoked the warning is typesafe, then (and only then) suppress the warning with an** **`@SuppressWarnings("unchecked")`** **annotation.**

> If you suppress warnings without first proving that the code is typesafe, you are giving yourself a false sense of security. The code may compile without emitting any warnings, but it can still throw a `ClassCastException` at runtime. 
>
> If, however, you ignore unchecked warnings that you know to be safe (instead of suppressing them), you won’t notice when a new warning crops up that represents a real problem. **The new warning will get lost amidst all the false alarms that you didn’t silence.**

**Always use the** **`SuppressWarnings`** **annotation on the smallest scope possible.** Typically this will be a **variable declaration** or a **very short method** or **constructor**. Never use `SuppressWarnings` on an entire class.

> If you find yourself using the `SuppressWarnings` annotation on a method or constructor that’s more than one line long, you may be able to move it onto **a local variable declaration**. You may have to declare a new local variable, but it’s worth it.

**Every time you use a `@SuppressWarnings("unchecked")` annotation, add a comment saying why it is safe to do so. **This will help others understand the code, and more importantly, it will decrease the odds that someone will modify the code so as to make the computation unsafe.

#### Example

Consider the `toArray` method, which comes from `ArrayList`:

```java
public <T> T[] toArray(T[] a) {
  if (a.length < size)
 	 return (T[]) Arrays.copyOf(elements, size, a.getClass()); System.arraycopy(elements, 0, a, 0, size);
  if (a.length > size)
    a[size] = null;
	return a;
}
```

If you compile `ArrayList`, the method generates this warning:

```java
ArrayList.java:305: warning: [unchecked] unchecked cast
    return (T[]) Arrays.copyOf(elements, size, a.getClass());
                               ^
    required: T[]
    found:    Object[]
```

**Solution**: declare a local variable to hold the return value and annotate its declaration, like so:

```java
// Adding local variable to reduce scope of @SuppressWarnings
public <T> T[] toArray(T[] a) {
   if (a.length < size) {
    	// This cast is correct because the array we're creating // is of the same type as the one passed in, which is T[]. 
      @SuppressWarnings("unchecked") T[] result =
         (T[]) Arrays.copyOf(elements, size, a.getClass());
      return result;
   }
   System.arraycopy(elements, 0, a, 0, size);
   if (a.length > size)
       a[size] = null;
   return a;
}
```

## Summary

In summary, unchecked warnings are important. Don’t ignore them. Every unchecked warning represents the potential for a `ClassCastException` at runtime. Do your best to eliminate these warnings. 

If you can’t eliminate an unchecked warning and you can prove that the code that provoked it is typesafe, suppress the warning with a `@SuppressWarnings("unchecked")` annotation in the narrowest possible scope. Record the rationale for your decision to suppress the warning in a comment.