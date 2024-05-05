---
title: cpp算法相关写法总结
categories:
  - [Algorithm]
---

## 输入

### 输入一行字符串到一个字符数组

例：

`abc def ghi` 到 `char str[1000]`

写法：

```cpp
char str[1000];
gets(str);
```

https://www.programiz.com/cpp-programming/library-function/cstdio/gets

### 用scanf读入一个字母 - 推荐读成字符串的形式

例：

`Q 2 3`

写法：

```cpp
char op[5];
int a, b;
scanf("%s%d%d", op, &a, &b);
// op[0]即为所需字母
```

原因：

如果用 `%c` 读op，容易读到空格回车等其他字符，很麻烦

但用 `%s` 会自动过滤空格回车（读到空格回车就停了）

## 输出

## 其他

### 比较字符串是否相等

```cpp
#include<string.h>

相等：!strcmp(op, "PM")
```

### memset

头文件：cstring

#### 0

```
memset(array, 0, sizeof(array))
```

#### -1

```
memset(array, -1, sizeof(array))
```

#### 无穷大

```
int类型：常采用0x3f3f3f3f来作为无穷大

memset(array, 0x3f, sizeof(array))
为数组设初值为0x3f3f3f3f，因为这个数的每个字节都是0x3f
```

### debug

1. cout
2. Segment Fault的时候一段一段删代码
