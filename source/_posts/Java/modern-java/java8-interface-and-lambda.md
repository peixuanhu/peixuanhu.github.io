---
title: Java 8-interface and lambda
categories:
  - [Java, Modern Java]
---

# Java 8 新特性

## Interface

### Default methods for Interfaces

Java 8 支持通过`default`关键字向接口添加非抽象方法实现。

目的：解决接口的修改与现有的实现不兼容的问题。

> Java 8 之前，`Interface` 修改的时候，实现它的类也必须跟着改。因为必须override所有抽象方法。

例：

```java
interface Formula{

    double calculate(int a);

    default double sqrt(int a) {
        return Math.sqrt(a);
    }

}
```

实现`Formula`接口的类无需override sqrt方法。

### Funtional Interfaces

定义：**仅仅只包含一个抽象方法,但是可以有多个非抽象方法(也就是上面提到的默认方法)的接口。**

目的：为了友好支持Lambda。函数式接口可以被隐式转换为 lambda 表达式。

> `java.lang.Runnable` 与 `java.util.concurrent.Callable` 是函数式接口最典型的两个例子。

Java 8 增加了一种特殊的注解`@FunctionalInterface`，但是这个注解通常不是必须的(某些情况建议使用)，只要接口只包含一个抽象方法，虚拟机会自动判断该接口为函数式接口。一般建议在接口上使用`@FunctionalInterface` 注解进行声明，这样的话，编译器如果发现你标注了这个注解的接口有多于一个抽象方法的时候会报错。

## Lambda

### 例子

在老版本的 Java 中是这样给字符串列表排序的：

```java
List<String> names = Arrays.asList("peter", "anna", "mike", "xenia");

Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return b.compareTo(a);
    }
});
```

只需要给静态方法`Collections.sort` 传入一个 List 对象以及一个比较器来按指定顺序排列。通常做法都是创建一个匿名的比较器对象然后将其传递给 `sort` 方法。

在 Java 8 中你就没必要使用这种传统的匿名对象的方式了，Java 8 提供了更简洁的语法，lambda 表达式：

```java
Collections.sort(names, (String a, String b) -> {
    return b.compareTo(a);
});
```

对于函数体只有一行代码的，你可以去掉大括号{}以及 return 关键字：

```java
Collections.sort(names, (String a, String b) -> b.compareTo(a));
```

但是你还可以写得更短点：

```java
names.sort((a, b) -> b.compareTo(a));
```

List 类本身就有一个 `sort` 方法。并且 Java 编译器可以自动推导出参数类型，所以你可以不用再写一次类型。
