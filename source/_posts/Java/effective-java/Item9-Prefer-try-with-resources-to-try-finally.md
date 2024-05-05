---
title: Effective Java-9. Prefer try-with-resources to try-finally
categories:
  - [Java, Effective Java]
---

# Item 9: Prefer *try-with-resources* to *try-finally*

## Background

The Java libraries include many resources that must be closed manually by invok- ing a close method. Examples include InputStream, OutputStream, and java.sql.Connection.

Closing resources is often overlooked by clients, with predictably dire performance consequences. While many of these resources use finalizers as a safety net, finalizers don’t work very well (Item 8).

## Solution

### 1. try-finally

Historically, a try-finally statement was the best way to guarantee that a resource would be closed properly, even in the face of an exception or return.

```java
// try-finally - No longer the best way to close resources!
static String firstLineOfFile(String path) throws IOException {
  BufferedReader br = new BufferedReader(new FileReader(path));
  try {
       return br.readLine();
   } finally {
       br.close();
   }
}
```

But it gets worse when you add a second resource:

```java
// try-finally is ugly when used with more than one resource!
 static void copy(String src, String dst) throws IOException {
     InputStream in = new FileInputStream(src);
     try {
         OutputStream out = new FileOutputStream(dst);
         try {
             byte[] buf = new byte[BUFFER_SIZE];
             int n;
             while ((n = in.read(buf)) >= 0)
                 out.write(buf, 0, n);
         } finally {
             out.close();
         }
     } finally {
         in.close();
     }
 }
```

#### Disadvantages

- It's complicated when dealing with multiple resources. It may be hard to believe, but even good programmers got this wrong most of the time.

- The code in both the try block and the finally block is capable of throwing exceptions. There is no record of the first exception in the exception stack trace, which can greatly complicate debugging in real systems—usually it’s the first exception that you want to see in order to diagnose the problem.

  > For example, in the *firstLineOfFile* method, the call to readLine could throw an exception due to a failure in the underlying physical device, and the call to close could then fail for the same reason. Under these circumstances, the second exception completely obliterates the first one.

### 2. try-with-resources

All of these problems were solved in one fell swoop when Java 7 introduced the try-with-resources statement.

To be usable with this construct, a resource must implement the **AutoCloseable** interface, which consists of a single void-returning close method.

> Many classes and interfaces in the Java libraries and in third-party libraries now implement or extend AutoCloseable. If you write a class that represents a resource that must be closed, your class should implement AutoCloseable too.

Examples:

```java
// try-with-resources - the the best way to close resources!
 static String firstLineOfFile(String path) throws IOException {
     try (BufferedReader br = new BufferedReader(
             new FileReader(path))) {
         return br.readLine();
     }
 }

// try-with-resources on multiple resources - short and sweet
static void copy(String src, String dst) throws IOException {
  try (InputStream in = new FileInputStream(src);
      OutputStream out = new FileOutputStream(dst)) {
    byte[] buf = new byte[BUFFER_SIZE];
    int n;
    while ((n = in.read(buf)) >= 0)
      out.write(buf, 0, n);
  }
}
```

Not only are the try-with-resources versions shorter and more **readable** than the originals, but they provide far better **diagnostics**.

> Consider the *firstLineOfFile* method. If exceptions are thrown by both the readLine call and the (invisible) close, **the latter exception is *suppressed* in favor of the former**. In fact, multiple exceptions may be suppressed in order to preserve the exception that you actually want to see.
>
> These suppressed exceptions are not merely discarded; they are printed in the stack trace with a notation saying that they were suppressed. You can also **access** them programmatically with the **getSuppressed** method, which was added to Throwable in Java 7.

#### catch clauses

You can put catch clauses on try-with-resources statements, just as you can on regular try-finally statements. This allows you to handle exceptions without sullying your code with another layer of nesting.

Here’s a version our firstLineOfFile method that does not throw exceptions, but takes a default value to return if it can’t open the file or read from it:

```java
// try-with-resources with a catch clause
static String firstLineOfFile(String path, String defaultVal) {
   try (BufferedReader br = new BufferedReader(
           new FileReader(path))) {
       return br.readLine();
   } catch (IOException e) {
       return defaultVal;
   }
}
```

## Summary

**Always use try-with-resources** in preference to try-finally when working with **resources that must be closed**.

The resulting code is shorter and clearer, and the exceptions that it generates are more useful. The try-with-resources statement makes it easy to write correct code using resources that must be closed, which was practically impossible using try-finally.

