---
title: Java集合概览
categories:
  - [Java, Collection]
---

![Java集合框架.png](https://s2.loli.net/2023/09/18/NSXU8CnfhvgZopd.png)

Java 集合， 也叫作容器，主要是由两大接口派生而来：

-  `Collection`接口，主要用于存放单一元素
  - `List`接口(对付顺序的好帮手): 存储的元素是有序的、可重复的。
    - `ArrayList`：`Object[]` 数组
    - `Vector`：`Object[]` 数组
    - `LinkedList`：双向链表(JDK1.6 之前为循环链表，JDK1.7 取消了循环)
  - `Set`接口(注重独一无二的性质): 存储的元素不可重复的。
    - `HashSet`(无序，唯一): 基于 `HashMap` 实现的，底层采用 `HashMap` 来保存元素
    - `LinkedHashSet`: `LinkedHashSet` 是 `HashSet` 的子类，并且其内部是通过 `LinkedHashMap` 来实现的。有点类似于我们之前说的 `LinkedHashMap` 其内部是基于 `HashMap` 实现一样，不过还是有一点点区别的
    - `TreeSet`(有序，唯一): 红黑树(自平衡的排序二叉树)
  - `Queue`接口(实现排队功能的叫号机): 按特定的排队规则来确定先后顺序，存储的元素是有序的、可重复的。
    - `PriorityQueue`: `Object[]` 数组来实现二叉堆
    - `ArrayQueue`: `Object[]` 数组 + 双指针
-  `Map` 接口(用 key 来搜索的专家): 使用键值对（key-value）存储，类似于数学上的函数 y=f(x)，"x" 代表 key，"y" 代表 value，key 是无序的、不可重复的，value 是无序的、可重复的，每个键最多映射到一个值。
  - `HashMap`：JDK1.8 之前 `HashMap` 由数组+链表组成的，数组是 `HashMap` 的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）。JDK1.8 以后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间
  - `LinkedHashMap`：`LinkedHashMap` 继承自 `HashMap`，所以它的底层仍然是基于拉链式散列结构即由数组和链表或红黑树组成。另外，`LinkedHashMap` 在上面结构的基础上，增加了一条双向链表，使得上面的结构可以保持键值对的插入顺序。同时通过对链表进行相应的操作，实现了访问顺序相关逻辑。
  - `Hashtable`：数组+链表组成的，数组是 `Hashtable` 的主体，链表则是主要为了解决哈希冲突而存在的
  - `TreeMap`：红黑树（自平衡的排序二叉树）

## 如何选用集合？

根据集合的特点来选择合适的集合。比如：

- 我们需要根据键值获取到元素值时就选用 `Map` 接口下的集合，需要排序时选择 `TreeMap`,不需要排序时就选择 `HashMap`,需要保证线程安全就选用 `ConcurrentHashMap`。
- 我们只需要存放元素值时，就选择实现`Collection` 接口的集合，需要保证元素唯一时选择实现 `Set` 接口的集合比如 `TreeSet` 或 `HashSet`，不需要就选择实现 `List` 接口的比如 `ArrayList` 或 `LinkedList`，然后再根据实现这些接口的集合的特点来选用。

## 为什么要选用集合？

相较于数组，Java 集合的优势在于它们的**大小可变、支持泛型、具有内建算法**等。总的来说，Java 集合提高了数据的存储和处理灵活性，可以更好地适应现代软件开发中多样化的数据需求，并支持高质量的代码编写。