---
title: Effective Java-11. Always override hashCode when you override equals
categories:
  - [Java, Effective Java]
---

# Item 11: Always override *hashCode* when you override *equals*

## General contract for *hashCode*

- When the *hashCode* method is invoked on an object repeatedly during **an execution of an application**, it **must consistently return the same value**, provided no information used in equals comparisons is modified. This value need not remain consistent from one execution of an application to another.
- If two objects are **equal** according to the *equals(Object)* method,then calling ***hashCode*** on the two objects must produce the **same integer result**.
- If two objects are unequal according to the *equals(Object)* method,it is *not* required that calling hashCode on each of the objects must produce distinct results. However, the programmer should be aware that producing distinct results for unequal objects may improve the performance of hash tables.

**The key provision that is violated when you fail to override** ***hashCode*** **is the second one: equal objects must have equal hash codes.** Two distinct instances may be logically equal according to a class’s equals method, but to Object’s *hashCode* method, they’re just two objects with nothing much in common. Therefore, Object’s *hashCode* method returns two seemingly random numbers instead of two equal numbers as required by the contract.

## The recipe to implement a good *hashCode* method

1. Declare an int variable named `result`, and initialize it to the hash code `c` for the **first significant field** in your object, as computed in step 2.a. (Recall from Item 10 that a significant field is a field that affects equals comparisons.)

2. For every **remaining significant field** `f` in your object, do the following: 

   a. Compute an int hash code `c` for the field:

   1. If the field is of a primitive type, compute `Type.hashCode(f)`, where Type is the boxed primitive class corresponding to `f`’ s type.
   2. If the field is an **object reference** and this class’s *equals* method compares the field by recursively invoking *equals*, **recursively invoke hashCode on the field**.

      If a more complex comparison is required, compute a “canonical representation” for this field and invoke hashCode on the canonical representation. If the value of the field is null, use 0 (or some other constant, but 0 is traditional).
   3. If the field is an array, treat it as if each significant element were a separate field. That is, compute a hash code for each significant element by applying these rules recursively, and combine the values per step 2.b. If the array has no significant elements, use a constant, preferably not 0. If all elements are significant, use `Arrays.hashCode`.

   b. Combine the hash code c computed in step 2.a into result as follows: 

   ```java
   result = 31 * result + c;
   ```

   > Makes the result depend on the **order** of the fields, yielding a much better hash function if the class has multiple similar fields.

3. Return result.

Example for the `PhoneNumber` class:

```java
// Typical hashCode method
@Override
public int hashCode() {
  int result = Short.hashCode(areaCode);
  result = 31 * result + Short.hashCode(prefix);
  result = 31 * result + Short.hashCOde(lineNum);
  return result;
}
```

> Exclude *derived fields* from the hash code computation.

> While the recipe in this item yields reasonably good hash functions, they are not state-of-the-art. They are comparable in quality to the hash functions found in the Java platform libraries’ value types and are adequate for most uses. If you have a bona fide need for hash functions **less likely to produce collisions**, see Guava’s `com.google.common.hash.Hashing` [Guava].

### Objects.hash static method

The *Objects* class has a static method that takes an arbitrary number of objects and returns a hash code for them. This method, named `hash`, lets you write **one-line** hashCode methods whose quality is comparable to those written according to the recipe in this item.

Unfortunately, they **run more slowly** because they entail array creation to pass a variable number of arguments, as well as boxing and unboxing if any of the arguments are of primitive type. This style of hash function is recommended for use only in situations where **performance is not critical**. 

Example:

```java
// One-line hashCode method - mediocre performance
@Override
public int hashCode() {
  return Objects.hash(lineNum, prefix, areaCode);
}
```

### Cache hashCode

If a class is **immutable** and the **cost of computing the hash code is significant**, you might consider **caching** the hash code in the object rather than recalculating it each time it is requested.

If you believe that most objects of this type will be used as **hash keys**, then you should calculate the hash code **when the instance is created**. Otherwise, you might choose to ***lazily initialize*** the hash code the first time *hashCode* is invoked. Some care is required to ensure that the class remains thread-safe in the presence of a lazily initialized field (Item 83).

```java
// hashCode method with lazily initialized cached hash Code
private int hashCode; // Automatically initialized to 0

@Override
public int hashCode() {
  int result = hashCode;
  if (result == 0) {
    result = Short.hashCode(areaCode);
    result = 31 * result + Short.hashCode(prefix);
    result = 31 * result + Short.hashCOde(lineNum);
    hashCode = result;
  }
  return result;
}
```

> Note that the initial value for the hashCode field (in this case, 0) should not be the hash code of a commonly created instance:

## Note

1. **Do not be tempted to exclude significant fields from the hash code computation to improve performance.** While the resulting hash function may run faster, its poor quality may degrade hash tables’ performance to the point where they become unusable.

   In particular, the hash function may be confronted with a large collection of instances that differ mainly in regions you’ve chosen to ignore. If this happens, the hash function will map all these instances to a few hash codes, and programs that should run in linear time will instead run in quadratic time.

2. **Don’t provide a detailed specification for the value returned by *hashCode*, so clients can’t reasonably depend on it; this gives you the flexibility to change it.** Many classes in the Java libraries, such as *String* and *Integer*, specify the exact value returned by their *hashCode* method as *a function of the instance value*. This is *not* a good idea but a mistake that we’re forced to live with: It impedes the ability to improve the hash function in future releases. If you leave the details unspecified and a flaw is found in the hash function or a better hash function is discovered, you can change it in a subsequent release.

## Summary

In summary, you *must* override *hashCode* every time you override *equals*, or your program will not run correctly. Your *hashCode* method must obey the **general contract specified in *Object*** and must do a reasonable job assigning unequal hash codes to unequal instances. This is easy to achieve, if slightly tedious, using the **recipe**.

As mentioned in Item 10, the ***AutoValue*** framework provides a fine alternative to writing *equals* and *hashCode* methods manually, and ***IDEs*** also provide some of this functionality.