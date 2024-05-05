---
title: Effective Java-13. Override clone judiciously
categories:
  - [Java, Effective Java]

---

# Item 13: Override *clone* judiciously

The `Cloneable` interface was intended as a *mixin interface* (Item 20) for classes to advertise that they permit cloning. Unfortunately, it fails to serve this purpose. Its primary flaw is that it lacks a `clone` method, and *Object*’s `clone` method is protected.

> You cannot, without resorting to *reflection* (Item 65), invoke clone on an object merely  because it implements `Cloneable`. Even a reflective invocation may fail, because there is no guarantee that the object has an accessible `clone` method.

## Effect of *Cloneable*

It determines the behavior of *Object*’s protected `clone` implementation: if a class implements `Cloneable`, *Object*’s `clone` method returns a **field-by-field copy of the object**; otherwise it throws `CloneNotSupportedException`.

> This is a highly atypical use of interfaces and not one to be emulated. Normally, implementing an interface says something about what a class can do for its clients. In this case, it modifies the behavior of a protected method on a superclass.

**In practice, a class implementing** **`Cloneable`** **is expected to provide a properly functioning public** **`clone`** **method.**

## General contract of *clone*

Creates and returns a copy of this object. The precise meaning of “copy” may depend on the class of the object.

- `x.clone() != x` will be true
- `x.clone().getclass() == x.getclass()` will be true
- `x.clone().equals(x)` will be true
- By convention, the object returned by this method should be obtained by calling `super.clone()`.
- By convention, **the returned object should be independent of the object being cloned**. To achieve this independence, it may be necessary to modify one or more fields of the object returned by super.clone before returning it.

> If a class’s clone method returns an instance that is *not* obtained by calling `super.clone` but by calling a constructor, the compiler won’t complain, but if a subclass of that class calls `super.clone`, the resulting object will have the wrong class.

## How to implment *clone* method

### 1. Immutable class

**Immutable classes should never provide a** **`clone`** **method** because it would merely encourage wasteful copying.

### 2. Class with no references to mutable state

First impelement `Cloneable` interface.

The `clone` method would look like:

```java
// Clone method for class with no references to mutable state
@Override
public PhoneNumber clone() {
  try {
    return (PhoneNumber) super.clone();
  } catch (CloneNotSupportedException e) {
    throw new AssertionError(); // Can't happen
  }
}
```

### 3. Class with fields that refer to mutable objects

Example: the *Stack* class in Item 7

If simply call `super.clone()`, its *elements* field will refer to the **same array** as the original *Stack* instance.

#### Solution

Call `clone` recursively on the *elements* array:

```java
// Clone method for class with no references to mutable state
@Override
public Stack clone() {
  try {
    Stack result = (Stack) super.clone();
    result.elements = elements.clone();
    return result;
  } catch (CloneNotSupportedException e) {
    throw new AssertionError();
  }
}
```

> 1. Note that we do not have to cast the result of elements.clone to Object[]. Calling clone on an array returns an array whose runtime and compile-time types are identical to those of the array being cloned. This is the preferred idiom to duplicate an array. In fact, **arrays are the sole compelling use of the clone facility**.
> 2. Note also that the earlier solution would not work if the elements field were **final** because clone would be prohibited from assigning a new value to the field. This is a fundamental problem: like serialization, **the** **`Cloneable`** **architecture is incompatible with normal use of final fields referring to mutable objects**. In order to make a class cloneable, it may be necessary to remove final modifiers from some fields.

### 4. Class with inner class

Example: HashTable implemented using its own linked list

#### Solution 1

Add a `deepCopy` method to the inner class.

```java
// Iteratively copy the linked list headed by this Entry
Entry deepCopy() {
  Entry result = new Entry(key, value, next);
  for (Entry p = result; p.next != null; p = p.next)
    p.next = new Entry(p.next.key, p.next.value, p.next.next);
  return result;
}
```

`clone` method of *HashTable* class:

```java
@Override
public HashTable clone() {
  try {
    HashTable result = (HashTable) super.clone();
    result.buckets = new Entry[buckets.length];
    for (int i = 0; i < buckets.length; i ++)
      if (buckets[i] != null)
        result.buckets[i] = buckets[i].deepCopy();
    return result;
  } catch (CloneNotSupportedException e) {
    throw new AssertionError();
  }
}
```

#### Solution 2

Call `super.clone`, set all of the fields in the resulting object to their **initial state**, and then call higher-level methods to regenerate the state of the original object.

In the case of our HashTable example, the buckets field would be initialized to a new bucket array, and the put(key, value) method would be invoked for each key-value mapping in the hash table being cloned.

> It is antithetical to the whole Cloneable architecture because it blindly overwrites the field-by-field object copy that forms the basis of the architecture.

## Note

1. Like a constructor, a `clone` method must **never invoke an overridable method on the clone** under construction (Item 19). If clone invokes a method that is overridden in a subclass, this method will execute before the subclass has had a chance to fix its state in the clone, quite possibly leading to corruption in the clone and the original.

   > Therefore, the put(key, value) method discussed in the previous paragraph should be either final or private. (If it is private, it is presumably the “helper method” for a nonfinal public method.)

2. *Object*’s `clone` method is declared to throw `CloneNotSupportedException`, but overriding methods need not. **Public** **`clone`** **methods should omit the** **throws** **clause**, as methods that don’t throw checked exceptions are easier to use (Item 71).

3. You have two choices when designing a **class for inheritance** (Item 19), but whichever one you choose, the class **should *not* implement `Cloneable`.**

   - You may choose to mimic the behavior of *Object* by **implementing a properly functioning protected `clone` method that is declared to throw `CloneNotSupportedException`.** This gives subclasses the freedom to implement Cloneable or not, just as if  dthey extended *Object* directly.

   - Alternatively, you may choose ***not* to implement a working `clone` method**, and to **prevent subclasses from implementing** one, by providing the following **degenerate `clone`** implementation:

     ```java
     // clone method for extendable class not supporting Cloneable
     @Override
     protected final Object clone() throws CloneNotSupportedException {
       throw new CloneNotSupportedException();
     }
     ```
     

4. If you write a **thread-safe class** that implements Cloneable, remember that its **`clone` method must be properly synchronized**, just like any other method (Item 78). *Object*’s clone method is not synchronized.

#### Conclusion

To recap, all classes that implement `Cloneable` should override `clone` with a public method whose **return type is the class itself**. This method should first **call `super.clone`**, then **fix any fields** that need fixing.

Typically, this means copying any mutable objects that comprise the internal “deep structure” of the object and replacing the clone’s references to these objects with references to their copies.

If the class contains only primitive fields or references to immutable objects, then it is likely the case that no fields need to be fixed. There are exceptions to this rule. For example, a field representing a serial number or other unique ID will need to be fixed even if it is primitive or immutable.

## Better approaches to object copying

**A better approach to object copying is to provide a *copy constructor* or *copy factory*.**

A copy constructor is simply a constructor that takes a single argument whose type is the class containing the constructor, for example:

```java
// Copy constructor
public Yum(Yum yum) { ... };
```

A copy factory is the static factory (Item 1) analogue of a copy constructor:

```java
// Copy factory
public static Yum newInstance(Yum yum) { ... };
```

### Advantages

- they don’t rely on a risk-prone extralinguistic object creation mechanism;

- they don’t demand unenforceable adherence to thinly documented conventions;

- they don’t conflict with the proper use of **final fields**;

- they don’t throw unnecessary **checked exceptions**;

- they don’t require **type casts**.

- a copy constructor or factory can take **an argument** whose type is **an interface implemented by the class**.

  > For example, suppose you have a HashSet, s, and you want to copy it as a TreeSet. The clone method can’t offer this functionality, but it’s easy with a conversion constructor: new TreeSet<>(s).

## Summary

Given all the problems associated with `Cloneable`, new **interfaces** should not extend it, and new **extendable classes** should not implement it.

While it’s less harmful for **final classes** to implement `Cloneable`, this should be viewed as a performance optimization, reserved for the rare cases where it is justified (Item 67).

As a rule, *copy* functionality is best provided by **constructors or factories**.

> A notable exception to this rule is **arrays**, which are best copied with the `clone` method.