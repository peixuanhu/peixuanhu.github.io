---
title: Effective Java-17. Minimize mutability
categories:
  - [Java, Effective Java]
---

# Item 17: Minimize mutability

An immutable class is simply a class whose instances cannot be modified. All of the information contained in each instance is **fixed** for the lifetime of the object, so no changes can ever be observed.

> The Java platform libraries contain many immutable classes, including *String*, the *boxed primitive classes*, and *BigInteger* and *BigDecimal*.

## Rules for making a class immutable

1. **Don’t provide methods that modify the object’s state** (known as *mutators*).

2. **Ensure that the class can’t be extended.** This prevents careless or malicious subclasses from compromising the immutable behavior of the class by behaving as if the object’s state has changed.

   Preventing subclassing is generally accomplished by:

   1. Making the class final;

   2. Or making all of its constructors private or package-private and add public static factories in place of the public constructors (Item 1), which is more flexible.

      Example:

      ```java
      // Immutable class with static factories instead of constructors
      public class Complex {
        private final double re;
        private final double im;
        
        private Complex(double re, double im) {
          this.re = re;
      		this.im = im;
        }
        
        public static Complex valueOf(double re, double im) {
        	return new Complex(re, im);
        }
         ... // Remainder unchanged
      }
      ```

      

3. **Make all fields final.** This clearly expresses your intent in a manner that is enforced by the system. Also, it is necessary to ensure correct behavior if a reference to a newly created instance is passed from one thread to another without synchronization, as spelled out in the *memory model*.

4. **Make all fields private, use *accessor* to access them.** This prevents clients from obtaining access to mutable objects referred to by fields and modifying these objects directly.

   > While it is technically permissible for immutable classes to have public final fields containing primitive values or references to immutable objects, it is not recommended because it precludes changing the internal representation in a later release (Items 15 and 16).

5. **Ensure exclusive access to any mutable components.** If your class has any fields that refer to **mutable objects**, ensure that clients of the class *cannot* obtain references to these objects. Never initialize such a field to a client-provided object reference or return the field from an accessor.

   > Make *defensive copies* (Item 50) in constructors, accessors, and `readObject` methods (Item 88).

## Pros & cons of immutable objects

### Pros

1. **Immutable objects are simple.** An immutable object can be in exactly one state, the state in which it was created. If you make sure that all constructors establish class invariants, then it is guaranteed that these invariants will remain true for all time, with no further effort on your part or on the part of the programmer who uses the class.

   > Mutable objects, on the other hand, can have arbitrarily complex state spaces. If the documentation does not provide a precise description of the state transitions performed by *mutator* methods, it can be difficult or impossible to use a mutable class reliably.

2. **Immutable objects are inherently thread-safe; they require no synchronization.** They cannot be corrupted by multiple threads accessing them concurrently. This is far and away the easiest approach to achieve thread safety.

   > Since no thread can ever observe any effect of another thread on an immutable object, **immutable objects can be shared freely.** Immutable classes should therefore encourage clients to reuse existing instances wherever possible. One easy way to do this is to **provide public static final constants for commonly used values**. For example, the `Complex` class might provide these constants:
   >
   > ```java
   > public static final Complex ZERO = new Complex(0, 0);
   > public static final Complex ONE = new Complex(1, 0);
   > public static final Complex I = new Complex(0, 1);
   > ```
   >
   > An immutable class can provide **static factories** (Item 1) that cache frequently requested instances to avoid creating new instances when existing ones would do.

3. **Not only can you share immutable objects, but they can share their internals.** 

   > For example, the `BigInteger` class uses a sign-magnitude representation internally. The sign is represented by an int, and the magnitude is represented by an int array. The negate method produces a new BigInteger of like magnitude and opposite sign. It does not need to copy the array even though it is mutable; the newly created `BigInteger` **points to the same internal array as the original**.

4. **Immutable objects make great building blocks for other objects,** whether mutable or immutable. It’s much easier to maintain the invariants of a complex object if you know that its component objects will not change underneath it.

   > A special case of this principle is that immutable objects make great map keys and set elements: you don’t have to worry about their values changing once they’re in the map or set, which would destroy the map or set’s invariants.

5. **Immutable objects provide failure atomicity for free** (Item 76). Their state never changes, so there is no possibility of a temporary inconsistency.

### Cons

**The major disadvantage of immutable classes is that they require a separate object for each distinct value.** Creating these objects can be costly, especially if they are large.

The package-private mutable companion class approach works fine if you can accurately predict which complex operations clients will want to perform on your immutable class. If not, then your best bet is to provide a *public* mutable companion class.

> The main example of this approach in the Java platform libraries is the String class, whose mutable companion is `StringBuilder` (and its obsolete predecessor, `StringBuffer`).

## Note

1. If you choose to have your immutable class implement `Serializable` and it contains one or more fields that refer to mutable objects, you must provide an explicit `readObject` or `readResolve` method, or use the `ObjectOutputStream.writeUnshared` and `ObjectInputStream.readUnshared` methods, even if the default serialized form is acceptable. **Otherwise an attacker could create a mutable instance of your class.** This topic is covered in detail in Item 88.
2. **Constructors should create fully initialized objects with all of their invari- ants established.** Don’t provide a public initialization method separate from the constructor or static factory unless there is a *compelling* reason to do so. Similarly, don’t provide a “reinitialize” method that enables an object to be reused as if it had been constructed with a different initial state. Such methods generally provide little if any performance benefit at the expense of increased complexity.

## Summary

To summarize, resist the urge to write a setter for every getter. **Classes should be immutable unless there’s a very good reason to make them mutable.** Immutable classes provide many advantages, and their only disadvantage is the potential for performance problems under certain circumstances. You should provide a public mutable companion class for your immutable class *only* once you’ve confirmed that it’s necessary to achieve satisfactory performance (Item 67).

> You should always make small value objects, such as `PhoneNumber` and Complex, immutable. (There are several classes in the Java platform libraries, such as `java.util.Date` and `java.awt.Point`, that should have been immutable but aren’t.) You should seriously consider making larger value objects, such as `String` and `BigInteger`, immutable as well.

There are some classes for which immutability is impractical. **If a class cannot be made immutable, limit its mutability as much as possible.** Reducing the number of states in which an object can exist makes it easier to reason about the object and reduces the likelihood of errors. Therefore, make every field final unless there is a compelling reason to make it nonfinal. Combining the advice of this item with that of Item 15, your natural inclination should be to **declare every field `private final` unless there’s a good reason to do otherwise.**
