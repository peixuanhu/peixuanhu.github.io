---
title: Effective Java-22. Use interfaces only to define types
categories:
  - [Java, Effective Java]
---

# Item 22: Use interfaces only to define types

When a class implements an interface, the interface serves as a ***type* that can be used to refer to instances of the class**. That a class implements an interface should therefore say something about what a client can do with instances of the class. It is inappropriate to define an interface for any other purpose. 

## Inappropriate example - export constants

One kind of interface that fails this test is the so-called *constant interface*. Such an interface contains no methods; it consists solely of static final fields, each exporting a constant. Classes using these constants implement the interface to avoid the need to qualify constant names with a class name.

Example:

```java
// Constant interface antipattern - do not use!
public interface PhysicalConstants {
   // Avogadro's number (1/mol)
   static final double AVOGADROS_NUMBER   = 6.022_140_857e23;
   // Boltzmann constant (J/K)
   static final double BOLTZMANN_CONSTANT = 1.380_648_52e-23;
   // Mass of the electron (kg)
   static final double ELECTRON_MASS      = 9.109_383_56e-31;
}
```

**The constant interface pattern is a poor use of interfaces.**

- That a class uses some constants internally is an implementation detail. Implementing a constant interface causes this implementation detail to leak into the class’s exported API.

- Worse, it represents a commitment: if in a future release the class is modified so that it no longer needs to use the constants, it still must implement the interface to ensure binary compatibility.

- If a nonfinal class implements a constant interface, all of its subclasses will have their namespaces polluted by the constants in the interface.

### Solution

1. If the constants are **strongly tied** to an existing class or interface, you should **add them to the class or interface**.

   > For example, all of the boxed numerical primitive classes, such as Integer and Double, export MIN_VALUE and MAX_VALUE constants.

2. If the constants are best viewed as members of an enumerated type, you should export them with an ***enum type*** (Item 34).

3. Otherwise, you should export the constants with a noninstantiable ***utility class*** (Item 4).

   Utility class version of the `PhysicalConstants` example:

   ```java
   // Constant utility class
   package com.effectivejava.science;
   public class PhysicalConstants {
     private PhysicalConstants() { }  // Prevents instantiation
     public static final double AVOGADROS_NUMBER = 6.022_140_857e23;
     public static final double BOLTZMANN_CONST =1.380_648_52e-23;
     public static final double ELECTRON_MASS = 9.109_383_56e-31;
   }
   ```

   

## Summary

In summary, interfaces should be used only to define types. They should not be used merely to export constants.