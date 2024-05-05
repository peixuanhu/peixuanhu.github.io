---
title: Effective Java-12. Always override toString
categories:
  - [Java, Effective Java]
---

# Item 12: Always override *toString*

While *Object* provides an implementation of the *toString* method, the string that it returns is generally not what the user of your class wants to see. It consists of **the class name** followed by an “at” sign (**@**) and the unsigned hexadecimal representation of the **hash code**, for example, `PhoneNumber@163b91`.

The general contract for *toString* says:

- The returned string should be “a **concise but informative** representation that is easy for a person to read.”
- It is recommended that **all subclasses** override this method.

While it isn’t as critical as obeying the equals and hashCode contracts (Items 10 and 11), **providing a good** **toString** **implementation makes your class much more pleasant to use and makes systems using the class easier to debug**.

> The toString method is automatically invoked when an object is passed to **println**, **printf**, the string concatenation operator (**+**), or ***assert***, or is printed by a debugger. 

## Principle

**When practical, the** **toString** **method should return** **all** **of the interesting information contained in the object**. Ideally, the string should be self-explanatory.

> It is impractical if the object is large or if it contains state that is not conducive to string representation. Under these circumstances, *toString* should return a **summary**.

## Important decision

One important decision you’ll have to make when implementing a *toString* method is **whether to specify the format of the return value in the documentation**.

It is recommended that you do this for ***value classes***, such as phone number or matrix.

> If you specify the format, it’s usually a good idea to provide a matching **static factory** or **constructor**, so programmers can easily translate back and forth between the **object** and **its string representation**.

### Advantage

It serves as a standard, unambiguous, human-readable representation of the object. This representation can be used for input and output and in persistent human-readable data objects, such as CSV files.

### Disadvantage

Once you’ve specified it, you’re stuck with it for life, assuming your class is widely used. Programmers will write code to parse the representation, to generate it, and to embed it into persistent data. If you change the representation in a future release, you’ll break their code and data, and they will yowl.

By choosing not to specify a format, you preserve the flexibility to add information or improve the format in a subsequent release.

### Advice

1. Whether or not you decide to specify the format, **you should clearly document your intentions.**

   If you specify the format, you should do so precisely. Example:

   ```java
   /**
   * Returns the string representation of this phone number.
   * The string consists of twelve characters whose format is
   * "XXX-YYY-ZZZZ", where XXX is the area code, YYY is the
   * prefix, and ZZZZ is the line number. Each of the capital
   * letters represents a single decimal digit.
   *
   * If any of the three parts of this phone number is too small
   * to fill up its field, the field is padded with leading zeros.
   * For example, if the value of the line number is 123, the last * four characters of the string representation will be "0123".
   */
   @Override public String toString() {
      return String.format("%03d-%03d-%04d", areaCode, prefix, lineNum);
   }
   ```

   If you decide not to specify a format, the documentation comment should read something like this:

   ```java
   /**
   * Returns a brief description of this potion. The exact details * of the representation are unspecified and subject to change, * but the following may be regarded as typical:
   *
   * "[Potion #9: type=love, smell=turpentine, look=india ink]"
   */
   @Override public String toString() { ... }
   ```

2. Whether or not you specify the format, **provide programmatic access to the information contained in the value returned by *toString*.** 

   > For example, the PhoneNumber class should contain accessors for the area code, prefix, and line number.

   If you fail to do this, you *force* programmers who need this information to parse the string. Besides reducing performance and making unnecessary work for programmers, this process is error-prone and results in fragile systems that break if you change the format.

## Note

1. It **makes no sense** to write a *toString* method in a **static utility class** (Item 4). Nor should you write a *toString* method in most **enum types** (Item 34) because Java provides a perfectly good one for you. 

2. You **should** write a *toString* method in **any abstract class whose subclasses share a common string representation**.

   > For example, the toString methods on most collection implemen- tations are inherited from the abstract collection classes.

## Summary

To recap, override *Object*’s *toString* implementation in **every instantiable class you write**, unless a superclass has already done so. It makes classes much more pleasant to use and aids in debugging. The *toString* method should return a concise, useful description of the object, in an aesthetically pleasing format.
