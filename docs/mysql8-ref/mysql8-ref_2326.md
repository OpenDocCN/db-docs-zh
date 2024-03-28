> 原文：[`dev.mysql.com/doc/refman/8.0/en/problems-with-null.html`](https://dev.mysql.com/doc/refman/8.0/en/problems-with-null.html)

#### B.3.4.3 NULL 值的问题

对于 SQL 新手来说，`NULL`值的概念是一个常见的困惑源，他们经常认为`NULL`和空字符串`''`是一样的。事实并非如此。例如，以下语句完全不同：

```sql
mysql> INSERT INTO my_table (phone) VALUES (NULL);
mysql> INSERT INTO my_table (phone) VALUES ('');
```

两个语句都向`phone`列插入一个值，但第一个插入了一个`NULL`值，第二个插入了一个空字符串。第一个的含义可以被视为“电话号码未知”，第二个的含义可以被视为“已知此人没有电话，因此没有电话号码”。

为了处理`NULL`，可以使用`IS NULL`和`IS NOT NULL`运算符以及`IFNULL()`函数。

在 SQL 中，`NULL`值与任何其他值（甚至`NULL`）的比较永远不成立。包含`NULL`的表达式总是产生`NULL`值，除非在表达式中涉及的运算符和函数的文档中另有说明。以下示例中的所有列都返回`NULL`：

```sql
mysql> SELECT NULL, 1+NULL, CONCAT('Invisible',NULL);
```

要搜索`NULL`列值，不能使用`expr = NULL`测试。以下语句不返回任何行，因为`expr = NULL`对于任何表达式都不成立：

```sql
mysql> SELECT * FROM my_table WHERE phone = NULL;
```

要查找`NULL`值，必须使用`IS NULL`测试。以下语句展示了如何找到`NULL`电话号码和空电话号码：

```sql
mysql> SELECT * FROM my_table WHERE phone IS NULL;
mysql> SELECT * FROM my_table WHERE phone = '';
```

参见 Section 5.3.4.6, “Working with NULL Values”，获取更多信息和示例。

如果使用`MyISAM`、`InnoDB`或`MEMORY`存储引擎，可以在可能包含`NULL`值的列上添加索引。否则，必须声明带索引的列为`NOT NULL`，并且不能将`NULL`插入到该列中。

在使用`LOAD DATA`读取数据时，空白或缺失的列会被更新为`''`。要将`NULL`值加载到列中，请在数据文件中使用`\N`。在某些情况下也可以使用文字`NULL`。参见 Section 15.2.9, “LOAD DATA Statement”。

在使用`DISTINCT`、`GROUP BY`或`ORDER BY`时，所有`NULL`值被视为相等。

在使用`ORDER BY`时，`NULL`值会首先呈现，如果指定`DESC`以降序排序，则会呈现在最后。

聚合（分组）函数如`COUNT()`、`MIN()`和`SUM()`会忽略`NULL`值。唯一的例外是`COUNT(*)`，它计算行数而不是单独的列值。例如，以下语句产生两个计数。第一个是表中行数的计数，第二个是`age`列中非`NULL`值的计数：

```sql
mysql> SELECT COUNT(*), COUNT(age) FROM person;
```

对于某些数据类型，MySQL 以特殊方式处理`NULL`值。例如，如果将`NULL`插入具有`AUTO_INCREMENT`属性的整数或浮点列中，下一个序列中的数字将被插入。在某些条件下，如果将`NULL`插入`TIMESTAMP`列中，将插入当前日期和时间；这种行为部分取决于服务器 SQL 模式（参见第 7.1.11 节，“服务器 SQL 模式”）以及`explicit_defaults_for_timestamp`系统变量的值。
