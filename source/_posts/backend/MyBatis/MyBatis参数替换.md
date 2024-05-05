---
title: MyBatis中#{}和${}的区别是什么？
categories:
  - [Backend, MyBatis]
---

动态 sql 是 MyBatis 的主要特性之一，在 mapper 中定义的参数传到 xml 中之后，在查询之前 MyBatis 会对其进行动态解析。MyBatis 为我们提供了两种支持动态 sql 的语法：#{} 以及 ${}。

## 一句话回答

\#{}是预编译处理，${}是字符串替换。

## 详细解释

- MyBatis在处理#{}时，会将SQL中的#{}替换为?号，使用PreparedStatement的set方法来赋值；

- 而MyBatis在处理${}时，直接把${}替换成变量的值。

## 示例

```xml
<select id="selectPerson" parameterType="int" resultType="hashmap">
  SELECT * FROM PERSON WHERE ID = #{id}
</select>
```

这个语句名为 selectPerson，接受一个 int（或 Integer）类型的参数，并返回一个 HashMap 类型的对象，其中的键是列名，值便是结果行中的对应值。

注意参数符号：#{id}

这就告诉 MyBatis 创建一个预处理语句（PreparedStatement）参数，在 JDBC 中，这样的一个参数在 SQL 中会由一个“?”来标识，并被传递到一个新的预处理语句中，就像这样：

```java
// 近似的 JDBC 代码，非 MyBatis 代码...
String selectPerson = "SELECT * FROM PERSON WHERE ID = ?";
PreparedStatement ps = conn.prepareStatement(selectPerson);
ps.setInt(1,id);
```

## 扩展

### 防止恶意SQL注入示例

#### 示例1

```java
String sql = "select * from tb_name where name = '" + varname + "' and passwd = '" + varpasswd + "' ";
```

如果我们把`[ ' or'1'='1']`作为`varpasswd`传入进来.用户名随意,看看会成为什么?

```sql
select * from tb_name = '随意' and passwd = ' ' or '1'='1';
```

因为'1'='1'肯定成立,所以可以任何通过验证

#### 示例2

```sql
select * from ${tableName} where name = #{name}
```

在这个例子中，如果${tableName}传入`user; delete user; --`

则动态解析之后 sql 如下：

```sql
select * from user; delete user; -- where name = ?;
```

--之后的语句被注释掉，而原本查询用户的语句变成了查询所有用户信息+删除用户表的语句，会对数据库造成重大损伤，极大可能导致服务器宕机。

### 预编译

##### 定义

指的是数据库驱动在发送 sql 语句和参数给 DBMS 之前对 sql 语句进行编译，这样 DBMS 执行 sql 时，就不需要重新编译。

##### 为什么需要预编译?

预编译是提前对SQL语句进行预编译，而其后注入的参数将不会再进行SQL编译。我们知道，SQL注入是发生在编译的过程中，因为恶意注入了某些特殊字符，最后被编译成了恶意的执行操作。而预编译机制则可以很好的防止SQL注入。

JDBC 中使用对象 PreparedStatement 来抽象预编译语句，使用预编译。

1. **预编译阶段可以优化 sql 的执行**。
   - 预编译之后的 sql 多数情况下可以直接执行，DBMS 不需要再次编译，越复杂的sql，编译的复杂度将越大，预编译阶段可以合并多次操作为一个操作。

2. **预编译语句对象可以重复利用**。
   - 把一个 sql 预编译后产生的 PreparedStatement 对象缓存下来，下次对于同一个sql，可以直接使用这个缓存的 PreparedState 对象。

MyBatis 默认情况下，将对所有的 sql 进行预编译。

### MyBatis sql 动态解析

MyBatis 在调用 connection 进行 sql 预编译之前，会对sql语句进行动态解析，动态解析主要包含如下的功能：

- 占位符的处理

- 动态sql的处理

- 参数类型校验