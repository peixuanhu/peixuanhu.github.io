---
title: Effective Java-18. Favor composition over inheritance
categories:
  - [Java, Effective Java]
---

# Item 18: Favor *composition* over *inheritance*

Inheritance is a powerful way to achieve code reuse, but it is not always the best tool for the job. Used inappropriately, it leads to fragile software. It is safe to use inheritance within a package, where the subclass and the superclass implementations are under the control of the same programmers. It is also safe to use inheritance when extending classes specifically designed and documented for extension (Item 19). Inheriting from ordinary concrete classes across package boundaries, however, is dangerous.

**Unlike method invocation, inheritance violates encapsulation.** In other words, a subclass depends on the implementation details of its superclass for its proper function. The superclass’s implementation may change from release to release, and if it does, the subclass may break, even though its code has not been touched. As a consequence, a subclass must evolve in tandem with its superclass, unless the superclass’s authors have designed and documented it specifically for the purpose of being extended.

## Solution

### Forwarding  class

Instead of extending an existing class, give your new class **a private field that references an instance of the existing class**. This design is called ***composition*** because the existing class becomes a component of the new one.

Each instance method in the new class invokes the corresponding method on the contained instance of the existing class and returns the results. This is known as ***forwarding***, and the methods in the new class are known as *forwarding methods*.

> The resulting class will be rock solid, with no dependencies on the implementation details of the existing class. Even adding new methods to the existing class will have no impact on the new class.

### Wrapper class

A class that "wraps" another forwarding class instance and adds some new function is known as a *wrapper* class, This is also known as the *Decorator* pattern. 

> Sometimes the combination of composition and forwarding is loosely referred to as *delegation.* Technically it’s not delegation unless the wrapper object passes itself to the wrapped object.

The disadvantages of wrapper classes are few. One caveat is that ***wrapper* classes are not suited for use in *callback frameworks***, wherein objects pass self-references to other objects for subsequent invocations (“callbacks”). Because a wrapped object doesn’t know of its wrapper, it passes a reference to itself (this) and callbacks elude the wrapper. This is known as the *SELF problem*.

## Note

Inheritance is appropriate only in circumstances where the subclass really is a *subtype* of the superclass. In other words, a class *B* should extend a class *A* only if an **“is-a”** relationship exists between the two classes.

If you are tempted to have a class *B* extend a class *A*, ask yourself the question: **Is every *B* really an *A*?** If you cannot truthfully answer yes to this question, *B* should not extend *A*. If the answer is no, it is often the case that *B* should contain a private instance of *A* and expose a different API: *A* is not an essential part of *B*, merely a detail of its implementation.
