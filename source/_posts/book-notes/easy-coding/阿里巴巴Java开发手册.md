---
title: 阿里巴巴Java开发手册
categories:
- [Book Notes, Easy Coding]
---

# 1. 编程规约

## 1.1 命名风格

- POJO类中的任何布尔型变量都不要加`is`前缀，否则部分框架解析会引起序列化错误。

  > 数据库中需要加`is`前缀，所以要在`<result_map>`设置从`is_xxx`到`xxx`的映射关系

- 