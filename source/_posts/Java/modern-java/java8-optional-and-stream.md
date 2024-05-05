---
title: Java 8-optional and stream
categories:
  - [Java, Modern Java]
---

# Java 8 新特性

## Optional

Optional 是一个简单的容器，其值可能是 null 或者不是 null。它是用于防止 NullPointerException 的工具。

有了Optional，我们可以安心地进行链式调用，而不是一层层判断是否为空。

### 用法

```java
// of()：为非null的值创建一个Optional
Optional<String> optional = Optional.of("bam"); // ofNullable(): 为可为空的值创建一个Optional
// isPresent()：如果值存在返回true，否则返回false
optional.isPresent();           // true
// get()：如果Optional有值则将其返回，否则抛出NoSuchElementException
optional.get();                 // "bam"
// orElse()：如果有值则将其返回，否则返回指定的其它值
optional.orElse("fallback");    // "bam"
// ifPresent()：如果Optional实例有值则为其调用consumer，否则不做处理
optional.ifPresent((s) -> System.out.println(s.charAt(0)));     // "b"
```

### 例子

```java
public static String getChampionName(Competition comp) throws IllegalArgumentException {
    if (comp != null) {
        CompResult result = comp.getResult();
        if (result != null) {
            User champion = result.getChampion();
            if (champion != null) {
                return champion.getName();
            }
        }
    }
    throw new IllegalArgumentException("The value of param comp isn't available.");
}
```

用 Optional 改写：

```java
public static String getChampionName(Competition comp) throws IllegalArgumentException {
    return Optional.ofNullable(comp)
            .map(Competition::getResult)  // 相当于c -> c.getResult()，下同
            .map(CompResult::getChampion)
            .map(User::getName)
            .orElseThrow(()->new IllegalArgumentException("The value of param comp isn't available."));
}
```

## Stream

Java 8 扩展了集合类，可以通过 `Collection.stream()` 或者 `Collection.parallelStream()` 来创建一个 Stream。

> 跟Spark写法很像

### filter

intermediate operation

> foreach is a terminal operation

```java
stringCollection
  .stream()
  .filter((s) -> s.startsWith("a"))
  .forEach(System.out::println);
```



### sorted

intermediate operation

可以指定自定义的 Comparator。

```java
stringList
  .stream()
  .sorted()
  .forEach(System.out::println);
```

排序只创建了一个排列好后的 Stream，而不会影响原有的数据源，排序之后原数据 stringList 是不会被修改的。

### map

intermediate operation

将元素根据指定的 Function 接口来依次将元素转成另外的对象。

```java
stringList
  .stream()
  .map(String::toUpperCase)
  .sorted((a, b) -> b.compareTo(a))
  .forEach(System.out::println);
```

### match

terminal operation

Stream 提供了多种匹配操作(anyMatch, allMatch, nonMatch)，允许检测指定的 Predicate 是否匹配整个 Stream。返回一个 boolean 类型的值。

```java
boolean anyStartsWithA =
  stringList
    .stream()
    .anyMatch((s) -> s.startsWith("a"));

boolean allStartsWithA =
  stringList
    .stream()
    .allMatch((s) -> s.startsWith("a"));


boolean noneStartsWithZ =
  stringList
    .stream()
    .noneMatch((s) -> s.startsWith("z"));
```



### count

terminal operation

返回 Stream 中元素的个数，**返回值类型是 long**。

```java
long startsWithB =
  stringList
    .stream()
    .filter((s) -> s.startsWith("b"))
    .count();
```

### reduce

terminal operation

允许通过指定的函数来将 stream 中的多个元素规约为一个元素，规约后的结果是通过 Optional 接口表示的。

```java
Optional<String> reduced =
  stringList
    .stream()
    .sorted()
    .reduce((s1, s2) -> s1 + "#" + s2);

reduced.ifPresent(System.out::println);
```

