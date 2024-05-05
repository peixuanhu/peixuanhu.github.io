---
title: Effective Java-8. Avoid finalizers and cleaners
categories:
  - [Java, Effective Java]
---

# Item 8: Avoid finalizers and cleaners

**Finalizers are unpredictable, often dangerous, and generally unnecessary.**

**Cleaners (Java 9) are less dangerous than finalizers, but still unpredictable, slow, and generally unnecessary.**

## Disadvantages

### 1. There is no guarantee they’ll be executed (promptly)

#### never do anything time-critical in a finalizer or cleaner

It can take arbitrarily long between the time that an object becomes unreachable and the time its finalizer or cleaner runs. This means that you should **never do anything time-critical in a finalizer or cleaner.**

> For example, it is a grave error to depend on a finalizer or cleaner to close files because open file descriptors are a limited resource. If many files are left open as a result of the system’s tardiness in running finalizers or cleaners, a program may fail because it can no longer open files.

The promptness with which finalizers and cleaners are executed is primarily a function of the *garbage collection algorithm*, which varies widely across implementations. The behavior of a program that depends on the promptness of finalizer or cleaner execution may likewise vary.

> It is entirely possible that such a program will run perfectly on the JVM on which you test it and then fail miserably on the one favored by your most important customer.

#### never depend on a finalizer or cleaner to update persistent state

Not only does the specification provide no guarantee that finalizers or cleaners will run promptly; it provides **no guarantee that they’ll run at all**. It is entirely possible, even likely, that a program terminates without running finalizer or cleaner on some objects that are no longer reachable. As a consequence, you should **never depend on a finalizer or cleaner to update persistent state.**

> For example, depending on a finalizer or cleaner to release a persistent lock on a shared resource such as a database is a good way to bring your entire distributed system to a grinding halt.

### 2. An uncaught exception thrown during finalization is ignored

Another problem with finalizers is that an uncaught exception thrown during finalization is ignored, and finalization of that object terminates. Uncaught exceptions can leave other objects in a corrupt state.

If another thread attempts to use such a corrupted object, arbitrary nondeterministic behavior may result. Normally, an uncaught exception will terminate the thread and print a stack trace, but not if it occurs in a finalizer—it won’t even print a warning.

> Cleaners do not have this problem because a library using a cleaner has control over its thread.

### 3. There is a severe performance penalty for using finalizers and cleaners

It is about 50 times slower to create and destroy objects with finalizers compared to using *try-with-resources*. This is primarily because finalizers inhibit efficient garbage collection.

Cleaners are comparable in speed to finalizers if you use them to clean all instances of the class, but cleaners are much faster if you use them only as a **safety net**.

### 4. Security problem: finalizers **open your class up to** *finalizer attacks*

> The idea behind a finalizer attack is simple: If an exception is thrown from a constructor or its serialization equivalents—the *readObject* and *readResolve* methods (Chapter 12)—the finalizer of a malicious subclass can run on the partially constructed object that should have “died on the vine.”(半途夭折) This finalizer can record a reference to the object in a static field, preventing it from being garbage collected. Once the malformed object has been recorded, it is a simple matter to invoke arbitrary methods on this object that should never have been allowed to exist in the first place. **Throwing an exception from a constructor should be sufficient to prevent an object from coming into existence; in the presence of finalizers, it is not.** 

Final classes are immune to finalizer attacks because no one can write a malicious subclass of a final class. **To protect nonfinal classes from finalizer attacks, write a final** **finalize** **method that does nothing.**

## Advantages

### 1. Can act as a safety net in case the owner of a resource neglects to call its close method

While there’s no guarantee that the cleaner or finalizer will run promptly (or at all), it is better to free the resource late than never if the client fails to do so. If you’re considering writing such a safety-net finalizer, think long and hard about whether the protection is **worth the cost**.

> Some Java library classes, such as FileInputStream, FileOutputStream, ThreadPoolExecutor, and java.sql.Connection, have finalizers that serve as safety nets.

### 2. Concerns objects with *native peers*

> A native peer is a native (non-Java) object to which a normal object delegates via native methods. 

Because a native peer is not a normal object, the garbage collector doesn’t know about it and can’t reclaim it when its Java peer is reclaimed.

A cleaner or finalizer may be an appropriate vehicle for this task, assuming the performance is acceptable and the native peer holds no critical resources. If the performance is unacceptable or the native peer holds resources that must be reclaimed promptly, the class should have a *close* method, as described earlier.

## Solution

What should you do instead of writing a finalizer or cleaner for a class whose objects encapsulate resources that require termination, such as files or threads?

Just **have your class implement** **AutoCloseable**, and require its clients to invoke the close method on each instance when it is no longer needed, typically using try-with-resources to ensure termination even in the face of exceptions (Item 9).

> One detail worth mentioning is that the instance must keep track of whether it has been closed: **the close method must record in a field that the object is no longer valid**, and other methods must check this field and throw an **IllegalStateException** if they are called after the object has been closed.

## Summary

In summary, don’t use cleaners, or in releases prior to Java 9, finalizers, except as a safety net or to terminate noncritical native resources. Even then, beware the indeterminacy and performance consequences.