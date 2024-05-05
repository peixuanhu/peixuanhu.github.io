---
title: Effective Java-20. Prefer interfaces to abstract classes
categories:
  - [Java, Effective Java]
---

# Item 20: Prefer interfaces to abstract classes

Java has two mechanisms to define a type that permits multiple implementations: **interfaces** and **abstract classes**. Since the introduction of *default methods* for interfaces in Java 8, both mechanisms allow you to provide implementations for some instance methods. 

A major difference is that to implement the type defined by an abstract class, a class must be a subclass of the abstract class. Because Java permits only **single inheritance**, this restriction on abstract classes severely **constrains their use as type definitions**. Any class that defines all the required methods and obeys the general contract is permitted to implement an interface, regardless of where the class resides in the class hierarchy.

## Advantages of interfaces

### 1. Easy to retrofit

**Existing classes can easily be retrofitted to implement a new interface.** All you have to do is to add the required methods, if they don’t yet exist, and to add an implements clause to the class declaration.

> Existing classes **cannot**, in general, be retrofitted to extend a new **abstract class**. If you want to have two classes extend the same abstract class, you have to place it high up in the type hierarchy where it is an ancestor of both classes. Unfortunately, this can cause great collateral damage to the type hierarchy, forcing all descendants of the new abstract class to subclass it, whether or not it is appropriate.

### 2. Ideal for defining mixins

**Interfaces are ideal for defining mixins.** An interface is called a mixin because it allows the optional functionality to be “mixed in” to the type’s primary functionality.

> Loosely speaking, a *mixin* is a type that a class can implement in addition to its “primary type,” to declare that it provides some optional behavior.

> Abstract classes can’t be used to define mixins for the same reason that they can’t be retrofitted onto existing classes: a class cannot have more than one parent, and there is no reasonable place in the class hierarchy to insert a mixin.

### 3. Nonhierarchical

**Interfaces allow for the construction of nonhierarchical type frameworks.**

*Type* hierarchies are great for organizing some things, but other things don’t fall neatly into a rigid hierarchy.

You don’t always need this level of flexibility, but when you do, interfaces are a lifesaver.

> The alternative is a bloated class hierarchy containing a separate class for every supported combination of attributes. If there are $n$ attributes in the type system, there are $2^n$ possible combinations that you might have to support. This is what’s known as a *combinatorial explosion*. Bloated class hierarchies can lead to bloated classes with many methods that differ only in the type of their arguments because there are no types in the class hierarchy to capture common behaviors.

### 4. Safe and powerful to enhance functionality

**Interfaces enable safe, powerful functionality enhancements** via the *wrapper class* idiom (Item 18). If you use abstract classes to define types, you leave the programmer who wants to add functionality with no alternative but inheritance. The resulting classes are less powerful and more fragile than wrapper classes.

## Skeletal implementation

When there is an **obvious implementation** of an interface method in terms of other interface methods, consider providing implementation assistance to programmers in the form of a **default method**.

### Limits of default methods

1. Although many interfaces specify the behavior of ***Object* methods** such as `equals` and `hashCode`, you are not permitted to provide default methods for them.

2. Also, interfaces are not permitted to contain **instance fields or nonpublic static members** (with the exception of private static methods).

3. Finally, you can’t add default methods to an interface that **you don’t control**.

### Solution

You can, however, combine the advantages of interfaces and abstract classes by providing an abstract ***skeletal implementation class*** to go with an interface.

- The *interface* defines the **type**, perhaps providing some **default methods**;

- The *skeletal implementation class* implements the **remaining non-primitive interface methods** atop the primitive interface methods.

  > This is the *Template Method* pattern.

Extending a skeletal implementation takes most of the work out of implementing an interface.

By convention, skeletal implementation classes are called **Abstract*Interface***, where *Interface* is the name of the interface they implement. For example, the Collections Framework provides a skeletal implementation to go along with each main collection interface: `AbstractCollection`, `AbstractSet`, `AbstractList`, and `AbstractMap`.

### Advantages

The beauty of skeletal implementation classes is that they provide all of the implementation assistance of abstract classes without imposing the severe constraints that abstract classes impose when they serve as type definitions. For most implementors of an interface with a skeletal implementation class, extending this class is the obvious choice, but it is strictly optional. If a class cannot be made to extend the skeletal implementation, the class can always implement the interface directly. The class still benefits from any default methods present on the interface itself.

Furthermore, the skeletal implementation can still aid the implementor’s task. **The class implementing the interface** can **forward invocations of interface methods** to a **contained instance of a private inner class that extends the skeletal implementation**. This technique, known as *simulated multiple inheritance*, is closely related to the wrapper class idiom (Item 18). It provides many of the benefits of multiple inheritance, while avoiding the pitfalls.

### How to write a skeletal implementation

- First, study the interface and decide which methods are the **primitives** in terms of which the others can be implemented. These primitives will be the **abstract methods in your skeletal implementation**.

- Next, provide **default methods** in the interface for all of the methods that **can be implemented directly atop the primitives**, but recall that you may not provide default methods for *Object* methods such as `equals` and `hashCode`.
  - If the primitives and default methods cover the interface, you’re done, and have no need for a skeletal implementation class.
  - Otherwise, write a class declared to implement the interface, with **implementations of all of the remaining interface methods**. The class may contain any nonpublic fields ands methods appropriate to the task.

Example: `Map.Entry`

```java
// Skeletal implementation class
public abstract class AbstractMapEntry<K,V> implements Map.Entry<K,V> {
  // Entries in a modifiable map must override this method
  @Override
  public V setValue(V value) {
     throw new UnsupportedOperationException();
  }
  
  // Implements the general contract of Map.Entry.equals
  @Override
  public boolean equals(Object o) {
    if (o == this)
     return true;
    if (!(o instanceof Map.Entry))
     return false;
    Map.Entry<?, ?> e = (Map.Entry) o;
    return Objects.equals(e.getKey(), getKey())
      && Objects.equals(e.getValue(), getValue());
  }
  
  // Implements the general contract of Map.Entry.hashCode
  @Override
  public int hashCode() {
     return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
  }
  
  @Override
  public String toString() {
		return getKey() + "=" + getValue();
  }
}
```

The obvious primitives are `getKey`, `getValue`, and (optionally) `setValue`. The interface specifies the behavior of `equals` and `hashCode`, and there is an obvious implementation of `toString` in terms of the **primitives**.

Since you are not allowed to provide default implementations for the *Object* methods, all implementations are placed in the skeletal implementation class.

> Note that this skeletal implementation could not be implemented in the `Map.Entry` **interface** or as a **subinterface** because **default methods are not permitted to override *Object* methods** such as `equals`, `hashCode`, and `toString`.

### Note

1. Because skeletal implementations are designed for inheritance, **good documentation is absolutely essential in a skeletal implementation,** whether it consists of default methods on an interface or a separate abstract class.
2. A minor variant on the skeletal implementation is the *simple implementation,* exemplified by `AbstractMap.SimpleEntry`. It differs in that it isn’t abstract: it is the simplest possible working implementation.

## Summary

To summarize, an interface is generally the best way to define a type that permits multiple implementations. If you export a nontrivial interface, you should strongly consider providing a **skeletal implementation** to go with it.

To the extent possible, you should provide the skeletal implementation via **default methods** on the interface so that all implementors of the interface can make use of it. That said, restrictions on interfaces typically mandate that a skeletal implementation take the form of an abstract class (not a subinterface).