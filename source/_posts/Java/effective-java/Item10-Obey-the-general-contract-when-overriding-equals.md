---
title: Effective Java-10. Obey the general contract when overriding equals
categories:
  - [Java, Effective Java]
---

# Item 10: Obey the general contract when overriding *equals*

## Conditions for **not** overriding the *equals* method

- **Each instance of the class is inherently unique.** This is true for classes such as **Thread** that represent active entities rather than values. The equals implementation provided by Object has exactly the right behavior for these classes.

- **There is no need for the class to provide a “logical equality” test.** For example, java.util.regex.Pattern could have overridden equals to check whether two Pattern instances represented exactly the same regular expression, but the designers didn’t think that clients would need or want this functionality. Under these circumstances, the *equals* implementation inherited from *Object* is ideal.

- **A super class has already overridden *equals*, and the superclass behavior is appropriate for this class.** For example, most *Set* implementations inherit their equals implementation from *AbstractSet*, *List* implementations from *AbstractList*, and *Map* implementations from *AbstractMap*.

- **The class is private or package-private,and you are certain that its *equals* method will never be invoked.** If you are extremely risk-averse, you can override the equals method to ensure that it isn’t invoked accidentally:

  ```java
  @Override
  public boolean equals(Object o) {
    throw new AssertionError(); // Method is never called
  }
  ```

  

## Overriding the *equals* method

Example:

```java
// Adds a value component without violating the equals contract
public class ColorPoint {
  private final Point point;
  private final Color color;
  
  public ColorPoint(int x, int y, Color color) {
     point = new Point(x, y);
     this.color = Objects.requireNonNull(color);
  }
  
  /**
   * Returns the point-view of this color point.
   */
  public Point asPoint() {
	  return point;
  }
  
  @Override
  public boolean equals(Object o) {
   if (!(o instanceof ColorPoint))
    return false;
   ColorPoint cp = (ColorPoint) o;
   return cp.point.equals(point) && cp.color.equals(color);
	}
  ...  // Remainder omitted
}
```

“Favor composition over inheritance.”

### When?

When a class has a notion of *logical equality* that differs from mere object identity and a superclass has not already overridden equals. This is generally the case for *value classes.*

> A value class is simply a class that represents a value, such as Integer or String.

A programmer who compares references to value objects using the *equals* method expects to find out whether they are **logically equivalent**, not whether they refer to the same object.

Not only is overriding the equals method necessary to satisfy programmer expectations, it enables instances to serve as **map keys or set elements** with predictable, desirable behavior.

> One kind of value class that does *not* require the *equals* method to be overridden is a class that uses **instance control** (Item 1) to ensure that **at most one object exists with each value**. *Enum* types (Item 34) fall into this category. For these classes, logical equality is the same as object identity, so Object’s *equals* method functions as a logical equals method.

### General contract of overriding *equals*

- ***Reflexivity***: For any non-null reference value x, **x.equals(x)** must return **true**.
- ***Symmetry***: For any non-null reference values x and  y, **x.equals(y)** must return true if and only if **y.equals(x)** returns true.
- ***Transitivity***: For any non-null reference values x, y, z, if **x.equals(y)**  returns true and **y.equals(z)** returns true, then **x.equals(z)** must return true.
- ***Consistency***: For any non-null reference values x and y, **multiple invocations** of x.equals(y) must consistently return true or consistently return false, provided no information used in equals comparisons is modified.
- ***Non-nullity***: For any non-null reference value x, **x.equals(null) must return false.**

  Type check is enough, no need to test "if o == null". The *instanceof* operator is specified to return false if its first operand is null, regardless of what type appears in the second operand.

### Recipe for a high-quality *equals* method

1. **Use the** **==** **operator to check if the argument is a reference to this object.** If so, return true. This is just a performance optimization but one that is worth doing if the **comparison is potentially expensive**.

2. **Use the *instanceof* operator to check if the argument has the correct type.** If not, return false. Typically, the correct type is the **class** in which the method occurs.

   > Occasionally, it is some interface implemented by this class. Use an interface if the class implements an interface that refines the equals contract to permit comparisons across classes that implement the interface. Collection interfaces such as Set, List, Map, and Map.Entry have this property.

3. **Cast the argument to the correct type.** Because this cast was preceded by an instanceof test, it is guaranteed to succeed.

4. **For each “significant” field in the class, check if that field of the argument matches the corresponding field of this object.** If all these tests succeed, return true; otherwise, return false. If the type in Step 2 is an interface, you must access the argument’s fields via interface methods; if the type is a class, you may be able to access the fields directly, depending on their accessibility.

   For primitive fields whose type is not float or double, use the == operator for comparisons.

   For object reference fields, call the equals method recursively.

   For float fields, use the static *Float.compare(float, float)* method; and for double fields, use *Double.compare(double, double)*. The special treatment of float and double fields is made necessary by the existence of Float.NaN, -0.0f and the analogous double values.

   For array fields, apply these guidelines to each element. If every element in an array field is significant, use one of the Arrays.equals methods.

   > Some object reference fields may **legitimately contain null**. To avoid the possibility of a NullPointerException, check such fields for equality using the static method **Objects.equals(Object, Object)**.

5. The performance of the equals method may be affected by the **order** in which fields are compared. For best performance, you should first compare fields that are **more likely to differ, less expensive to compare**, or, ideally, both.

   You must **not compare fields that are not part of an object’s logical state**, such as lock fields used to synchronize operations.

   You need **not compare *derived fields***, which can be calculated from “significant fields,” but doing so may improve the performance of the equals method. If a derived field amounts to a summary description of the entire object, comparing this field will save you the expense of comparing the actual data if the comparison fails.

6. Write unit tests to check, unless you used *AutoValue* to generate your equals method, in which case you can safely omit the tests.

### Final caveats

-  **Always override hashCode when you override *equals***(Item11).

- **Don’t try to be too clever.** If you simply test fields for equality, it’s not hard to adhere to the equals contract. If you are overly aggressive in searching for equivalence, it’s easy to get into trouble. It is generally a bad idea to take any form of aliasing into account. For example, the File class shouldn’t attempt to equate symbolic links referring to the same file. Thankfully, it doesn’t.

- **Don’t substitute another type for *Object* in the *equals* declaration.**Itisnot uncommon for a programmer to write an equals method that looks like this and then spend hours puzzling over why it doesn’t work properly:

  ```java
  // Broken - parameter type must be Object!
  public boolean equals(MyClass o) {
     ...
  }
  ```

  The problem is that this method does not *override* Object.equals, whose argument is of type Object, but *overloads* it instead (Item 52).

  Consistent use of the *@Override* annotation, will prevent you from making this mistake (Item 40), it won't compile and the error message will tell you exactly what is wrong.

### Tools

Writing and testing equals (and hashCode) methods is tedious, and the resulting code is mundane. An excellent alternative to writing and testing these methods manually is to use **Google’s open source AutoValue framework**, which automatically generates these methods for you, triggered by a single annotation on the class . In most cases, the methods generated by AutoValue are essentially identical to those you’d write yourself.

**IDEs**, too, have facilities to generate equals and hashCode methods, but the resulting source code is more verbose and less readable than code that uses AutoValue, does not track changes in the class automatically, and therefore requires testing. That said, having IDEs generate equals (and hashCode) methods is generally preferable to implementing them manually because IDEs do not make careless mistakes, and humans do.

## Summary

In summary, don’t override the *equals* method unless you have to: **in many cases, the implementation inherited from Object does exactly what you want.**

If you do override equals, make sure to compare all of the class’s significant fields and to compare them in a manner that preserves all **five provisions of the equals contract.
