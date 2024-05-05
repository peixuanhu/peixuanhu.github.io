---
title: Java集合-Set
categories:
  - [Java, Collection]
---

## 比较 HashSet、LinkedHashSet 和 TreeSet 三者的异同

- `HashSet`、`LinkedHashSet` 和 `TreeSet` 都是 `Set` 接口的实现类，都能保证元素唯一，并且都**不是线程安全的**。

- `HashSet`、`LinkedHashSet` 和 `TreeSet` 的主要区别在于底层数据结构不同。
  - `HashSet` 的底层数据结构是哈希表（基于 `HashMap` 实现）。
  - `LinkedHashSet` 的底层数据结构是链表和哈希表，元素的插入和取出顺序满足 FIFO。
  - `TreeSet` 底层数据结构是红黑树，元素是有序的，排序的方式有自然排序和定制排序。

- 底层数据结构不同又导致这三者的应用场景不同。
  - `HashSet` 用于不需要保证元素插入和取出顺序的场景。
  - `LinkedHashSet` 用于保证元素的插入和取出顺序满足 FIFO 的场景。
  - `TreeSet` 用于支持对元素自定义排序规则的场景。

## Comparable 和 Comparator 的区别

`Comparable` 接口和 `Comparator` 接口都是 Java 中用于排序的接口，它们在实现类对象之间比较大小、排序等方面发挥了重要作用：

- `Comparable` 接口出自`java.lang`包，它有一个 `compareTo(Object obj)`方法用来排序（拿自己和其他对象比较）
- `Comparator`接口出自 `java.util` 包，它有一个`compare(Object obj1, Object obj2)`方法用来排序（传入两个对象比较）

一般我们需要对一个集合使用自定义排序时，我们就要**重写`compareTo()`方法或`compare()`方法**。

- 当我们需要对某一个集合实现两种排序方式，比如一个 `song` 对象中的歌名和歌手名分别采用一种排序方法的话：
  - 我们可以重写`compareTo()`方法和使用自制的`Comparator`方法来实现歌名排序和歌星名排序
  - 或者以两个 `Comparator` 来实现歌名排序和歌星名排序，我们只能使用两个参数版的 `Collections.sort(collection, comparator)`.

## 无序性和不可重复性的含义

- 无序性不等于随机性 ，无序性是指存储的数据在底层数组中并非按照数组索引的顺序添加 ，而是根据数据的哈希值决定的。

- 不可重复性是指添加的元素按照 `equals()` 判断时 ，返回 false，需要同时重写 `equals()` 方法和 `hashCode()` 方法。

##  HashSet 如何检查重复?

先检查`hashcode`，如果有重复的再调用`equals()`。

>当你把对象加入`HashSet`时，`HashSet` 会先计算对象的`hashcode`值来判断对象加入的位置，同时也会与其他加入的对象的 `hashcode` 值作比较，如果没有相符的 `hashcode`，`HashSet` 会假设对象没有重复出现。但是如果发现有相同 `hashcode` 值的对象，这时会调用`equals()`方法来检查 `hashcode` 相等的对象是否真的相同。如果两者相同，`HashSet` 就不会让加入操作成功。

在 JDK1.8 中，`HashSet`的`add()`方法只是简单的调用了`HashMap`的`put()`方法，并且判断了一下返回值以确保是否有重复元素。直接看一下`HashSet`中的源码：

```java
// Returns: true if this set did not already contain the specified element
// 返回值：当 set 中没有包含 add 的元素时返回真
public boolean add(E e) {
        return map.put(e, PRESENT)==null;
}
```

而在`HashMap`的`putVal()`方法中也能看到如下说明：

```java
// Returns : previous value, or null if none
// 返回值：如果插入位置没有元素返回null，否则返回上一个元素
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
...
}
```

也就是说，在 JDK1.8 中，实际上无论`HashSet`中是否已经存在了某元素，`HashSet`都会直接插入，只是会在`add()`方法的返回值处告诉我们插入前是否存在相同元素。