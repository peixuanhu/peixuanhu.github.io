---
title: Effective Java-16. In public classes, use accessor methods, not public fields
categories:
  - [Java, Effective Java]
---

# Item 16: In public classes, use accessor methods, not public fields

## Public class

**If a class is accessible outside its package, provide accessor methods** to preserve the flexibility to change the class’s internal representation. If a public class exposes its data fields, all hope of changing its representation is lost because client code can be distributed far and wide.

```java
// Encapsulation of data by accessor methods and mutators
class Point {
  private double x;
  private double y;
  public Point(double x, double y) {
    this.x = x;
    this.y = y;
  }
  public double getX() { return x; }
  public double getY() { return y; }
  public void setX(double x) { this.x = x; }
  public void setY(double y) { this.y = y; }
}
```

> If **a class has no access control identifier**, it has default access control characteristics. This default access control stipulates that **the class can only be accessed and referenced by classes in the same package**, and cannot be used by classes in other packages, even if there are subclasses of this class in other packages. This access feature is also called package access (package private).

While it’s never a good idea for a public class to expose fields directly, it is less harmful if the fields are **immutable**. You can’t change the representation of such a class without changing its API, and you can’t take auxiliary actions when a field is read, but you can enforce invariants.

Example:

```java
// Public class with exposed immutable fields - questionable
public final class Time {
  private static final int HOURS_PER_DAY    = 24;
  private static final int MINUTES_PER_HOUR = 60;
  public final int hour;
  public final int minute;
  public Time(int hour, int minute) {
    if (hour < 0 || hour >= HOURS_PER_DAY)
      throw new IllegalArgumentException("Hour: " + hour);
    if (minute < 0 || minute >= MINUTES_PER_HOUR)
      throw new IllegalArgumentException("Min: " + minute);
    this.hour = hour;
    this.minute = minute;
	}
   ... // Remainder omitted
}
```

## Package-private / private nested class

However, **if a class is package-private or is a private nested class, there is nothing inherently wrong with exposing its data fields**—assuming they do an adequate job of describing the abstraction provided by the class.

This approach generates less visual clutter than the accessor-method approach, both in the class definition and in the client code that uses it.

> While the client code is tied to the class’s internal representation, this code is **confined to the package** containing the class. If a change in representation becomes desirable, you can make the change without touching any code outside the package. In the case of a private nested class, the scope of the change is further restricted to the enclosing class.

## Summary

In summary, **public classes should never expose mutable fields**. It is less harmful, though still questionable, for public classes to expose immutable fields.

It is, however, sometimes desirable for **package-private or private nested classes** to **expose fields**, whether mutable or immutable.