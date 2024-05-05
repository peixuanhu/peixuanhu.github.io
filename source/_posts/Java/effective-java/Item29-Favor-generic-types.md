---
title: Effective Java-29. Favor generic types
categories:
  - [Java, Effective Java]
---

# Item 29: Favor generic types

## Example

```java
// Object-based collection - a prime candidate for generics
public class Stack {
  private Object[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;
  
  public Stack() {
    elements = new Object[DEFAULT_INITIAL_CAPACITY];
  }
  
  public void push(Object e) { 
    ensureCapacity(); elements[size++] = e;
  }
  
  public Object pop() { 
    if (size == 0)
      throw new EmptyStackException();
    Object result = elements[--size];
    elements[size] = null; // Eliminate obsolete reference 
    return result;
  }
  
  public boolean isEmpty() {
     return size == 0;
  }
  
  private void ensureCapacity() {
     if (elements.length == size)
         elements = Arrays.copyOf(elements, 2 * size + 1);
  } 
}
```

This class should have been parameterized to begin with, but since it wasn’t, we can *generify* it after the fact.

```java
// Initial attempt to generify Stack - won't compile! 
public class Stack<E> {
  private E[] elements;
  private int size = 0;
  private static final int DEFAULT_INITIAL_CAPACITY = 16;
  
  public Stack() {
    elements = new E[DEFAULT_INITIAL_CAPACITY];
  }
  
  public void push(E e) { 
    ensureCapacity(); elements[size++] = e;
  }
  
  public E pop() { 
    if (size == 0)
      throw new EmptyStackException();
    E result = elements[--size];
    elements[size] = null; // Eliminate obsolete reference return result;
  }
  ... // no changes in isEmpty or ensureCapacity
}
```

You’ll generally get at least one error or warning:

```java
Stack.java:8: generic array creation
  elements = new E[DEFAULT_INITIAL_CAPACITY];
  					 ^
```

As explained in Item 28, you can’t create an array of a non-reifiable type, such as E. This problem arises every time you write a generic type that is backed by an array.

### Solution 1

The first way: directly circumvents the prohibition on generic array creation: create an array of `Object` and cast it to the generic array type. Now in place of an error, the compiler will emit a warning. This usage is legal, but it’s not (in general) typesafe:

```java
Stack.java:8: warning: [unchecked] unchecked cast
found: Object[], required: E[]
	elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
								^
```

> The compiler may not be able to prove that your program is typesafe, but you can. You must convince yourself that the unchecked cast will not compromise the type safety of the program. The array in question (elements) is stored in a private field and never returned to the client or passed to any other method. The only elements stored in the array are those passed to the push method, which are of type `E`, so the unchecked cast can do no harm.

Once you’ve proved that an unchecked cast is safe, suppress the warning in as narrow a scope as possible (Item 27). With the addition of an annotation to do this, Stack compiles cleanly, and you can use it without explicit casts or fear of a `ClassCastException`:

```java
// The elements array will contain only E instances from push(E). 
// This is sufficient to ensure type safety, but the runtime
// type of the array won't be E[]; it will always be Object[]! 
@SuppressWarnings("unchecked")
public Stack() {
   elements = (E[]) new Object[DEFAULT_INITIAL_CAPACITY];
}
```

### Solution 2

The second way to eliminate the generic array creation error in Stack is to change the type of the field elements from `E[]` to `Object[]`. If you do this, you’ll get a different error:

```java
Stack.java:19: incompatible types
found: Object, required: E
	E result = elements[--size];
										 ^
```

You can change this error into a warning by casting the element retrieved from the array to `E`, but you will get a warning:

```java
Stack.java:19: warning: [unchecked] unchecked cast
found: Object, required: E
	E result = (E) elements[--size]; 
												 ^
```

Because `E` is a non-reifiable type, there’s no way the compiler can check the cast at runtime. Again, you can easily prove to yourself that the unchecked cast is safe, so it’s appropriate to suppress the warning.

```java
// Appropriate suppression of unchecked warning
public E pop() {
 if (size == 0)
     throw new EmptyStackException();
	// push requires elements to be of type E, so cast is correct 
  @SuppressWarnings("unchecked") 
  E result = (E) elements[--size];
  elements[size] = null; // Eliminate obsolete reference
  return result;
}
```

## Comparison

Both techniques for eliminating the generic array creation have their adherents. 

- The first is more readable: the array is declared to be of type `E[]`, clearly indicating that it contains only `E` instances;

- The first is also more concise: in a typical generic class, you read from the array at many points in the code; 

- The first technique requires only a single cast (where the array is created), while the second requires a separate cast each time an array element is read. 

- However, the first causes *heap pollution* (Item 32): the runtime type of the array does not match its compile-time type (unless `E` happens to be `Object`). 

  > This makes some programmers sufficiently queasy that they opt for the second technique, though the heap pollution is harmless in this situation.

Overall, the first technique is preferable and more commonly used in practice. 

## Note

The great majority of generic types are like our `Stack` example in that their type parameters have no restrictions: you can create a `Stack<Object>`, `Stack<int[]>`, `Stack<List<String>>`, or `Stack` of any other object reference type. 

Note that **you can’t create a `Stack` of a primitive type**: trying to create a `Stack<int>` or `Stack<double>` will result in a compile-time error. This is a fundamental limitation of Java’s generic type system. You can work around this restriction by using boxed primitive types (Item 61).

## Summary

In summary, generic types are safer and easier to use than types that require casts in client code. When you design new types, make sure that they can be used without such casts. This will often mean making the types generic. 

If you have any existing types that should be generic but aren’t, generify them. This will make life easier for new users of these types without breaking existing clients (Item 26).