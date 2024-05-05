---
title: Effective Java-1. Consider static factory methods instead of constructors
categories:
  - [Java, Effective Java]
---

# Item 1: Consider static factory methods instead of constructors

## 优势

### 1. 静态工厂方法有名称

能够确切描述被返回的对象。

### 2. 不必在每次调用它们时都创建一个新对象

与享元模式类似。节约资源。

### 3. 可以返回原返回类型的任何子类型的对象

通过接口来引用被返回的对象，而不是通过它的实现类。

### 4. 所返回的对象的类可以随着每次调用而发生变化，取决于静态工厂方法的参数值

> 如：Enumset只有静态工厂方法，它返回两种子类之一的实例，具体取决于底层枚举类型的大小：
>
> - 如果它的元素小于等于64个，则像大多数枚举类型一样，返回一个RegularEnumset，用单个long支持；
> - 如果枚举类型有大于64个元素，工厂返回JumboEnumset，用一个long数组支持

### 5. 方法返回的对象所属的类，在编写包含该静态方法的类时可以不存在

Such flexible static factory methods form the basis of *service provider frameworks*, like the Java Database Connectivity API (JDBC).

> A service provider framework is a system in which providers implement a service, and the system makes the implementations available to clients, decoupling the clients from the implementations.

## 缺点

### 1. 类如果不含public/protected constructor，就不能被子类化

For example, it is impossible to subclass any of the convenience implementation classes in the Collections Framework.

### 2. 程序员很难发现静态工厂方法

以下是静态工厂方法的一些惯用名称：

- from
- of
- valueOf
- instance / getInstance
- create / newInstance
- getType: Type表示工厂方法所返回的对象类型
- newType
- type