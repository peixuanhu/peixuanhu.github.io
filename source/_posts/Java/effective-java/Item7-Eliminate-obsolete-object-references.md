---
title: Effective Java-7. Eliminate obsolete object references
categories:
  - [Java, Effective Java]
---

# Item 7: Eliminate obsolete object references

## Example of memory leak

```java
// Can you spot the "memory leak"?
public class Stack {
  private Object[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;
  public Stack() {
    elements = new Object[DEFAULT_INITIAL_CAPACITY];
  }
  public void push(Object e) {
    ensureCapacity();
    elements[size++] = e;
  }
  public Object pop() {
    if (size == 0)
      throw new EmptyStackException();
    return elements[--size];
  }
  /**
  * Ensure space for at least one more element, roughly
  * doubling the capacity each time the array needs to grow.
  */
  private void ensureCapacity() {
    if (elements.length == size)
      elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```



### Where is the memory leak? 

If a stack grows and then shrinks, the objects that were popped off the stack will not be garbage collected, even if the program using the stack has no more references to them. This is because the stack maintains **obsolete references** to these objects. An obsolete reference is simply a reference that will never be dereferenced again.

> The memory leak can silently manifest itself as reduced performance due to increased garbage collector activity or increased memory footprint. In extreme cases, such memory leaks can cause **disk paging** and even program failure with an **OutOfMemoryError**, but such failures are relatively rare.

### Fix for the memory leak

Fix: null out references once they become obsolete.

```java
public Object pop() {
  if (size == 0)
  	throw new EmptyStackException();
  Object result = elements[--size];
  elements[size] = null; // Eliminate obsolete reference
  return result;
}
```

## Nulling out obsolete references

### Advantages

- avoid memory leak
- if they are subsequently dereferenced by mistake, the program will immediately fail with a NullPointerException, rather than quietly doing the wrong thing. It is always beneficial to detect programming errors as quickly as possible.

### Note

**Nulling out object references should be the exception rather than the norm.** The best way to eliminate an obsolete reference is to let the variable that contained the reference fall out of scope. This occurs naturally if you define each variable in the narrowest possible scope (Item 57).

## Sources of memory leaks

###  1. A class manages its own memory

Whenever a class manages its own memory, the programmer should be alert for memory leaks. Whenever an element is freed, any object references contained in the element should be nulled out.

### 2. Caches

Once you put an object reference into a cache, it’s easy to forget that it’s there and leave it in the cache long after it becomes irrelevant.

#### Solutions

- Represent the cache as a **WeakHashMap**

  - Situation: an entry is relevant(有意义) exactly when there are references to its key outside of the cache, entries will be removed automatically after they become obsolete.

  - Remember that *WeakHashMap* is useful only if the desired lifetime of cache entries is determined by external references to the **key**, not the value.

- Use a background thread (perhaps a **ScheduledThreadPoolExecutor**) to cleanse the cache occasionally, or as a side effect of adding new entries to the cache(method **removeEldestEntry** of **LinkedHashMap**).

- For more sophisticated caches, you may need to use **java.lang.ref** directly.

### 3. Listeners and callbacks

If you implement an API where clients register callbacks but **don’t deregister them explicitly**, they will accumulate unless you take some action.

One way to ensure that callbacks are garbage collected promptly is to **store only weak references to them**, for instance, by storing them only as keys in a *WeakHashMap*.