---
title: Effective Java-21. Design interfaces for posterity
categories:
  - [Java, Effective Java]
---

# Item 21: Design interfaces for posterity

Prior to Java 8, it was impossible to add methods to interfaces without breaking existing implementations. If you added a new method to an interface, existing implementations would, in general, lack the method, resulting in a compile-time error.

In Java 8, the *default method* construct was added, with the intent of allowing the **addition of methods to existing interfaces**. But adding new methods to existing interfaces is fraught with risk.

The declaration for a default method includes a *default implementation* that is used by all classes that implement the interface but do not implement the default method. While the addition of default methods to Java makes it possible to add methods to an existing interface, **there is no guarantee that these methods will work in all preexisting implementations.** Default methods are “injected” into existing implementations without the knowledge or consent of their implementors. Before Java 8, these implementations were written with the tacit understanding that their interfaces would *never* acquire any new methods.

> Many new default methods were added to the core collection interfaces in Java 8, primarily to facilitate the use of *lambdas* (Chapter 6). The Java libraries’ default methods are high-quality general-purpose implementations, and in most cases, they work fine. But **it is not always possible to write a default method that maintains all invariants of every conceivable implementation.**

> Preexisting collection implementations that were not part of the Java platform did not have the opportunity to make analogous changes in lockstep with the interface change, and some have yet to do so.

**In the presence of default methods, existing implementations of an interface may compile without error or warning but fail at runtime.** A handful of the methods added to the collections interfaces in Java 8 are known to be susceptible, and a handful of existing implementations are known to be affected.

## Note

1. Using default methods to **add new methods to existing interfaces** should be avoided unless the need is critical, in which case you should think long and hard about whether an existing interface implementation might be broken by your default method implementation.

   Default methods are, however, extremely useful for **providing standard method implementations when an interface is created**, to ease the task of implementing the interface (Item 20).

2. It is also worth noting that default methods were **not** designed to support **removing methods from interfaces or changing the signatures of existing methods**. Neither of these interface changes is possible without breaking existing clients.

## Summary

Even though default methods are now a part of the Java platform, **it is still of the utmost importance to design interfaces with great care.** While default methods make it *possible* to add methods to existing interfaces, there is great risk in doing so.

> If an interface contains a minor flaw, it may irritate its users **forever**; if an interface is severely deficient, it may doom the API that contains it.

Therefore, it is critically important to **test each new interface before you release it**. Multiple programmers should implement each interface in different ways.

> At a minimum, you should aim for **three diverse implementations**. Equally important is to write **multiple client programs** that use instances of each new interface to perform various tasks.
>
> - This will go a long way toward ensuring that each interface **satisfies all of its intended uses**.
>
> - These steps will allow you to **discover flaws** in interfaces **before** they are released, when you can still correct them easily.
>
> While it may be possible to correct some interface flaws **after** an interface is released, **you cannot count on it**.