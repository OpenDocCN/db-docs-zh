# 17.20.4 InnoDB memcached 多个获取和范围查询支持

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-memcached-multiple-get-range-query.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-memcached-multiple-get-range-query.html)

`daemon_memcached`插件支持多个获取操作（在单个**memcached**查询中获取多个键值对）和范围查询。

#### 多个获取操作

在单个**memcached**查询中获取多个键值对的能力通过减少客户端和服务器之间的通信流量来提高读取性能。对于`InnoDB`，这意味着更少的事务和打开表操作。

以下示例演示了多个获取支持。该示例使用了创建新表和列映射中描述的`test.city`表。

```sql
mysql> USE test;
mysql> SELECT * FROM test.city;
+---------+-----------+-------------+---------+-------+------+--------+
| city_id | name      | state       | country | flags | cas  | expiry |
+---------+-----------+-------------+---------+-------+------+--------+
| B       | BANGALORE | BANGALORE   | IN      |     0 |    1 |      0 |
| C       | CHENNAI   | TAMIL NADU  | IN      |     0 |    0 |      0 |
| D       | DELHI     | DELHI       | IN      |     0 |    0 |      0 |
| H       | HYDERABAD | TELANGANA   | IN      |     0 |    0 |      0 |
| M       | MUMBAI    | MAHARASHTRA | IN      |     0 |    0 |      0 |
+---------+-----------+-------------+---------+-------+------+--------+
```

运行`get`命令以从`city`表中检索所有值。结果以键值对序列返回。

```sql
telnet 127.0.0.1 11211
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
get B C D H M
VALUE B 0 22
BANGALORE|BANGALORE|IN
VALUE C 0 21
CHENNAI|TAMIL NADU|IN
VALUE D 0 14
DELHI|DELHI|IN
VALUE H 0 22
HYDERABAD|TELANGANA|IN
VALUE M 0 21
MUMBAI|MAHARASHTRA|IN
END
```

在单个`get`命令中检索多个值时，您可以切换表（使用`@@*`containers.name`*`符号）以检索第一个键的值，但不能为后续键切换表。例如，此示例中的表切换是有效的：

```sql
get @@aaa.AA BB
VALUE @@aaa.AA 8 12
HELLO, HELLO
VALUE BB 10 16
GOODBYE, GOODBYE
END
```

尝试在同一个`get`命令中再次切换表以从不同表中检索键值不受支持。

通过多个获取操作检索的键的数量没有限制，但存储结果的内存限制为 128MB。

#### 范围查询

对于范围查询，`daemon_memcached`插件支持以下比较运算符：`<`，`>`，`<=`，`>=`。运算符前必须加上`@`符号。当范围查询找到多个匹配的键值对时，结果以键值对序列返回。

以下示例演示了范围查询支持。这些示例使用了创建新表和列映射中描述的`test.city`表。

```sql
mysql> SELECT * FROM test.city;
+---------+-----------+-------------+---------+-------+------+--------+
| city_id | name      | state       | country | flags | cas  | expiry |
+---------+-----------+-------------+---------+-------+------+--------+
| B       | BANGALORE | BANGALORE   | IN      |     0 |    1 |      0 |
| C       | CHENNAI   | TAMIL NADU  | IN      |     0 |    0 |      0 |
| D       | DELHI     | DELHI       | IN      |     0 |    0 |      0 |
| H       | HYDERABAD | TELANGANA   | IN      |     0 |    0 |      0 |
| M       | MUMBAI    | MAHARASHTRA | IN      |     0 |    0 |      0 |
+---------+-----------+-------------+---------+-------+------+--------+
```

打开 telnet 会话：

```sql
telnet 127.0.0.1 11211
Trying 127.0.0.1...
Connected to 127.0.0.1.
Escape character is '^]'.
```

要获取所有大于`B`的值，请输入`get @>B`：

```sql
get @>B
VALUE C 0 21
CHENNAI|TAMIL NADU|IN
VALUE D 0 14
DELHI|DELHI|IN
VALUE H 0 22
HYDERABAD|TELANGANA|IN
VALUE M 0 21
MUMBAI|MAHARASHTRA|IN
END
```

要获取所有小于`M`的值，请输入`get @<M`：

```sql
get @<M
VALUE B 0 22
BANGALORE|BANGALORE|IN
VALUE C 0 21
CHENNAI|TAMIL NADU|IN
VALUE D 0 14
DELHI|DELHI|IN
VALUE H 0 22
HYDERABAD|TELANGANA|IN
END
```

要获取所有小于或等于`M`的值，请输入`get @<=M`：

```sql
get @<=M
VALUE B 0 22
BANGALORE|BANGALORE|IN
VALUE C 0 21
CHENNAI|TAMIL NADU|IN
VALUE D 0 14
DELHI|DELHI|IN
VALUE H 0 22
HYDERABAD|TELANGANA|IN
VALUE M 0 21
MUMBAI|MAHARASHTRA|IN
```

要获取大于`B`但小于`M`的值，请输入`get @>B@<M`：

```sql
get @>B@<M
VALUE C 0 21
CHENNAI|TAMIL NADU|IN
VALUE D 0 14
DELHI|DELHI|IN
VALUE H 0 22
HYDERABAD|TELANGANA|IN
END
```

最多可以解析两个比较运算符，一个是'小于'（`@<`）或'小于或等于'（`@<=`）运算符，另一个是'大于'（`@>`）或'大于或等于'（`@>=`）运算符。任何额外的运算符都被视为键的一部分。例如，如果您发出带有三个运算符的`get`命令，则第三个运算符（`@>C`）被视为键的一部分，`get`命令将搜索小于`M`且大于`B@>C`的值。

```sql
get @<M@>B@>C
VALUE C 0 21
CHENNAI|TAMIL NADU|IN
VALUE D 0 14
DELHI|DELHI|IN
VALUE H 0 22
HYDERABAD|TELANGANA|IN
```
