---
title: Effective Java-15. Minimize the accessibility of classes and members
categories:
  - [Java, Effective Java]
---

# Item 15: Minimize the accessibility of classes and members

The single most important factor that distinguishes a well-designed component from a poorly designed one is the degree to which **the component hides its internal data and other implementation details from other components**.

A well-designed component **hides all its implementation details, cleanly separating its API from its implementation**. Components then **communicate only through their APIs** and are oblivious to each others’ inner workings. This concept, known as *information hiding* or ***encapsulation***, is a fundamental tenet of software design.

## The importance of *information hiding*

- It ***decouples*** the components that comprise a system, allowing them to be developed, tested, optimized, used, understood, and modified **in isolation**. This speeds up system development because components can be developed in parallel.

- It eases the burden of **maintenance** because components can be understood more quickly and debugged or replaced with little fear of harming other components.

  > Once a system is complete and profiling has determined which components are causing performance problems (Item 67), those components can be optimized without affecting the correctness of others.

- It increases software **reuse** because components that aren’t tightly coupled often prove useful in other contexts besides the ones for which they were developed.
- It decreases the risk in building large systems because individual components may prove successful even if the system does not.

## Approach to aid in information hiding - *access control*

The *access control* mechanism specifies the *accessibility* of classes, interfaces, and members. The accessibility of an entity is determined by the **location of its declaration** and by which, if any, of the **access modifiers** (`private`, `protected`, and `public`) is present on the declaration. Proper use of these modifiers is essential to information hiding.

### Specific rules for classes

The rule of thumb is simple: **make each class or member as inaccessible as possible.** In other words, **use the lowest possible access level** consistent with the proper functioning of the software that you are writing.

If a top-level class or interface can be made package-private, it should be. By making it **package-private**, you make it part of the implementation rather than the exported API, and **you can modify it, replace it, or eliminate it in a subsequent release without fear of harming existing clients**. If you make it public, you are obligated to support it forever to maintain compatibility.

If a package-private top-level class or interface is used by **only one class**, consider making the top-level class **a private static nested class** of the sole class that uses it (Item 24).

> But it is far more important to reduce the accessibility of a **gratuitously public class** than of a package-private top-level class: the public class is part of the package’s API, while the package-private top- level class is already part of its implementation.

### Specific rules for members

For members (fields, methods, nested classes, and nested interfaces), there are four possible access levels, listed here in order of increasing accessibility:

- **private**—The member is accessible only from the top-level class where it is declared.
- **package-private**—The member is accessible from any class in the package where it is declared. Technically known as *default* access, this is the access level you get if no access modifier is specified (except for interface members, which are public by default).
- **protected**—The member is accessible from subclasses of the class where it is declared (subject to a few restrictions [JLS, 6.6.2]) and from any class in the package where it is declared.
- **public**—The member is accessible from anywhere.

After carefully designing your class’s **public API**, your reflex should be to make all other members **private**. Only if another class in the same package really needs to access a member should you remove the private modifier, making the member package-private.

> These fields can, however, “leak” into the exported API if the class implements `Serializable` (Items 86 and 87).

#### Override

If a method **overrides a superclass method**, it **cannot** have a **more restrictive access level** in the subclass than in the superclass. This is necessary to ensure that an instance of the subclass is usable anywhere that an instance of the superclass is usable (the *Liskov substitution principle*).

#### Members of public classes

For members of public classes, a huge increase in accessibility occurs when the access level goes from **package-private** to **protected**. A protected member is part of the class’s exported API and must be supported forever. Also, a protected member of an exported class represents a public commitment to an implementa- tion detail (Item 19). **The need for protected members should be relatively rare.**

**Instance fields of public classes should rarely be public** (Item 16). If an instance field is **nonfinal** or is a reference to a mutable object, then by making it public, you **give up the ability to limit the values** that can be stored in the field. Also, you **give up the ability to take any action when the field is modified**, so classes with public mutable fields are **not** generally **thread-safe**.

> Even if a field is **final** and refers to an immutable object, by making it public you **give up the flexibility to switch to a new internal data representation** in which the field does not exist.

#### Array field

Note that a nonzero-length array is always mutable, so **it is wrong for a class to have a public static final array field, or an accessor that returns such a field.** If a class has such a field or accessor, clients will be able to modify the contents of the array. This is a frequent source of security holes:

```java
// Potential security hole!
public static final Things[] VALUES = { ... };
```

Beware of the fact that some IDEs generate accessors that return references to pri- vate array fields, resulting in exactly this problem.

##### Solution 1

You can make the public array private and add a public immutable list:

```java
 private static final Thing[] PRIVATE_VALUES = { ... };
 public static final List<Thing> VALUES =
     Collections.unmodifiableList(Arrays.asList(PRIVATE_VALUES));
```

##### Solution 2

Alternatively, you can make the array private and add a public method that returns a copy of a private array:

```java
private static final Thing[] PRIVATE_VALUES = { ... };
public static final Thing[] values() {
   return PRIVATE_VALUES.clone();
}
```

## Module

As of Java 9, there are two additional, implicit access levels introduced as part of the *module system*.

A module may explicitly export some of its packages via *export declarations* in its *module declaration* (which is by convention contained in a source file named `module-info.java`).

Public and protected members of unexported packages in a module are inaccessible outside the module; within the module, accessibility is unaffected by export declarations. Using the module system allows you to **share classes among packages within a module without making them visible to the entire world**.

## Summary

To summarize, you should reduce accessibility of program elements as much as possible (within reason).

After carefully designing a **minimal public API**, you should prevent any stray classes, interfaces, or members from becoming part of the API.

With the exception of public static final fields (which serve as constants), **public classes should have no public fields**. Ensure that objects referenced by public static final fields are immutable.