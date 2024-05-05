---
title: Effective Java-3. Enforce the singleton property with a private constructor or an enum type
categories:
  - [Java, Effective Java]
---

# Item 3: Enforce the singleton property with a private constructor or an enum type

## Background

A *singleton* is simply a class that is instantiated exactly once. Singletons typically represent either a stateless object such as a function (Item 24) or a system component that is intrinsically unique.

## Implementations

### Public field approach

```java
// Singleton with public final field
public class Elvis {
  public static final Elvis INSTANCE = new Elvis();
  private Elvis() { ... }
  
  public void leaveTheBuilding() { ... }
}
```

Advantages:

- the API makes it clear that the class is a singleton: the public static field is final, so it will always contain the same object reference.
- it's simpler.

> A privileged client can **invoke the private constructor reflectively** (Item 65) with the aid of the AccessibleObject.setAccessible method. If you need to defend against this attack, modify the **constructor** to make it **throw an exception** if it’s asked to create a second instance.

### Static factory approach

```java
// Singleton with static factory
public class Elvis {
  private static final Elvis INSTANCE = new Elvis();
  private Elvis() { ... }
  public static Elvis getInstance() { return INSTANCE; }
  
  public void leaveTheBuilding() { ... }
}
```

Advantages:

- it gives you the flexibility to change your mind about whether the class is a singleton without changing its API.
- you can write a *generic singleton factory* if your application requires it (Item 30).
- a *method reference* can be used as a supplier, for example `Elvis::instance` is a `Supplier<Elvis>`.

> Unless one of these advantages is relevant, the public field approach is preferable.

### Single-element enum approach

**A single-element enum type is often the best way to implement a singleton**.

This approach is similar to the public field approach, but it is more concise, provides the serialization machinery for free, and provides an **ironclad guarantee against multiple instantiation**, even in the face of **sophisticated serialization** or **reflection attacks**.

> Note that you **can’t** use this approach if your singleton must **extend a superclass other than Enum** (though you *can* declare an enum to implement interfaces).

```java
// Enum singleton - the preferred approach
public enum Elvis {
  INSTANCE;
  
  public void leaveTheBuilding() { ... }
}
```



## Note

To make a singleton class that uses either of **public field / static factory approaches** *serializable* (Chapter 12), it is not sufficient merely to add implements Serializable to its declaration.

To maintain the singleton guarantee, declare all instance fields transient and provide a *readResolve* method (Item 89). Otherwise, each time a serialized instance is deserialized, a new instance will be created, leading, in the case of our example, to spurious Elvis sightings.

```java
// readResolve method to preserve singleton property
private Object readResolve() {
  // Return the one true Elvis and let the garbage collector take care of the Elvis impersonator
  return INSTANCE;
}
```

