ConcurrentHashMap更新值：

```java
map.compute(word, (k, v) -> v == null ? 1 : v + 1);

// 或者用merge，可以对新键的值做初始化（第二个参数），但不能处理键
map.merge(word, 1L, Integer::sum);
```

ConcurrentHashMap中键和值都不能为null。
