---
title: Java 8-other new features
categories:
  - [Java, Modern Java]
---

# Java 8 新特性

## Method and Constructor References

Java 8 支持用`::`关键字传递方法（静态方法或对象方法都可以）或构造函数的引用。

### 构造函数引用例子

定义一个包含多个构造函数的简单类：

```java
class Person {
    String firstName;
    String lastName;

    Person() {}

    Person(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
}
```

接下来我们指定一个用来创建 Person 对象的对象工厂接口：

```java
interface PersonFactory<P extends Person> {
  P create(String firstName, String LastName);
}
```

使用构造函数引用来将他们关联起来，而不是手动实现一个完整的工厂：

```java
PersonFactory<Person> personFactory = Person::new;
Person person = personFactory.create("Peter", "Parker");
```

只需要使用 `Person::new` 来获取 Person 类构造函数的引用，Java 编译器会自动根据`PersonFactory.create`方法的参数类型来选择合适的构造函数。

## Map

Map 类型不支持 streams，不过 Map 提供了一些新的有用的方法来处理一些日常任务。

Map 接口本身没有可用的 `stream()`方法，但是你可以在键，值上创建专门的流或者通过 `map.keySet().stream()`,`map.values().stream()`和`map.entrySet().stream()`。

此外,Maps 支持各种新的和有用的方法来执行常见任务。

```java
Map<Integer, String> map = new HashMap<>();

for (int i = 0; i < 10; i++) {
    map.putIfAbsent(i, "val" + i);
}

map.forEach((id, val) -> System.out.println(val));//val0 val1 val2 val3 val4 val5 val6 val7 val8 val9
```

`putIfAbsent` 阻止我们在 null 检查时写入额外的代码;`forEach`接受一个 consumer 来对 map 中的每个元素操作。

此示例显示如何使用函数在 map 上计算代码：

```java
map.computeIfPresent(3, (num, val) -> val + num);
map.get(3);             // val33

map.computeIfPresent(9, (num, val) -> null);
map.containsKey(9);     // false

map.computeIfAbsent(23, num -> "val" + num);
map.containsKey(23);    // true

map.computeIfAbsent(3, num -> "bam");
map.get(3);             // val33
```

接下来展示如何在 Map 里删除一个键值全都匹配的项：

```java
map.remove(3, "val3");
map.get(3);             // val33
map.remove(3, "val33");
map.get(3);             // null
```

另外一个有用的方法：

```java
map.getOrDefault(42, "not found");  // not found
```

对 Map 的元素做合并也变得很容易了：

```java
map.merge(9, "val9", (value, newValue) -> value.concat(newValue));
map.get(9);             // val9
map.merge(9, "concat", (value, newValue) -> value.concat(newValue));
map.get(9);             // val9concat
```

Merge 做的事情是如果键名不存在则插入，否则对原键对应的值做合并操作并重新插入到 map 中。
