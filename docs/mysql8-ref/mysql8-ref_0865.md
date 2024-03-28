# 14.19.3 MySQL GROUP BY 处理

> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-by-handling.html`](https://dev.mysql.com/doc/refman/8.0/en/group-by-handling.html)

SQL-92 及更早版本不允许查询，其中选择列表、`HAVING`条件或`ORDER BY`列表引用未在`GROUP BY`子句中命名的非聚合列。例如，这个查询在标准 SQL-92 中是非法的，因为选择列表中的非聚合`name`列不出现在`GROUP BY`中：

```sql
SELECT o.custid, c.name, MAX(o.payment)
  FROM orders AS o, customers AS c
  WHERE o.custid = c.custid
  GROUP BY o.custid;
```

要使查询在 SQL-92 中合法，`name`列必须在选择列表中省略或在`GROUP BY`子句中命名。

SQL:1999 及更高版本允许这样的非聚合列，如果它们在功能上依赖于`GROUP BY`列，则可选择功能 T301 的可选特性：如果`name`和`custid`之间存在这样的关系，则查询是合法的。例如，如果`custid`是`customers`的主键，则会出现这种情况。

MySQL 实现了功能依赖的检测。如果启用了（默认情况下启用的）`ONLY_FULL_GROUP_BY` SQL 模式，MySQL 会拒绝查询，其中选择列表、`HAVING`条件或`ORDER BY`列表引用既不在`GROUP BY`子句中命名也不在功能上依赖于它们的非聚合列。

当启用 SQL `ONLY_FULL_GROUP_BY`模式时，MySQL 还允许在`GROUP BY`子句中未命名的非聚合列，前提是该列被限制为单个值，如下例所示：

```sql
mysql> CREATE TABLE mytable (
 ->    id INT UNSIGNED NOT NULL PRIMARY KEY,
 ->    a VARCHAR(10),
 ->    b INT
 -> );

mysql> INSERT INTO mytable
 -> VALUES (1, 'abc', 1000),
 ->        (2, 'abc', 2000),
 ->        (3, 'def', 4000);

mysql> SET SESSION sql_mode = sys.list_add(@@session.sql_mode, 'ONLY_FULL_GROUP_BY');

mysql> SELECT a, SUM(b) FROM mytable WHERE a = 'abc';
+------+--------+
| a    | SUM(b) |
+------+--------+
| abc  |   3000 |
+------+--------+
```

当使用`ONLY_FULL_GROUP_BY`时，`SELECT`列表中也可以有多个非聚合列。在这种情况下，每个这样的列必须在`WHERE`子句中限制为单个值，并且所有这些限制条件必须通过逻辑`AND`连接，如下所示：

```sql
mysql> DROP TABLE IF EXISTS mytable;

mysql> CREATE TABLE mytable (
 ->    id INT UNSIGNED NOT NULL PRIMARY KEY,
 ->    a VARCHAR(10),
 ->    b VARCHAR(10),
 ->    c INT
 -> );

mysql> INSERT INTO mytable
 -> VALUES (1, 'abc', 'qrs', 1000),
 ->        (2, 'abc', 'tuv', 2000),
 ->        (3, 'def', 'qrs', 4000),
 ->        (4, 'def', 'tuv', 8000),
 ->        (5, 'abc', 'qrs', 16000),
 ->        (6, 'def', 'tuv', 32000);

mysql> SELECT @@session.sql_mode;
+---------------------------------------------------------------+
| @@session.sql_mode                                            |
+---------------------------------------------------------------+
| ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ENGINE_SUBSTITUTION |
+---------------------------------------------------------------+

mysql> SELECT a, b, SUM(c) FROM mytable
 ->     WHERE a = 'abc' AND b = 'qrs';
+------+------+--------+
| a    | b    | SUM(c) |
+------+------+--------+
| abc  | qrs  |  17000 |
+------+------+--------+
```

如果禁用了`ONLY_FULL_GROUP_BY`，MySQL 对`GROUP BY`的标准 SQL 使用的扩展允许选择列表、`HAVING`条件或`ORDER BY`列表引用非聚合列，即使这些列在功能上不依赖于`GROUP BY`列。这导致 MySQL 接受前面的查询。在这种情况下，服务器可以自由选择每个组中的任何值，因此除非它们相同，否则所选的值是不确定的，这可能不是您想要的。此外，从每个组中选择值后，不能通过添加`ORDER BY`子句来影响。结果集排序发生在值被选择之后，`ORDER BY`不影响服务器选择每个组中的哪个值。禁用`ONLY_FULL_GROUP_BY`主要在您知道由于数据的某些属性，`GROUP BY`中未命名的每个非聚合列的所有值对于每个组都相同时才有用。

你可以通过使用 `ANY_VALUE()` 引用非聚合列来达到相同效果，而不禁用 `ONLY_FULL_GROUP_BY`。

以下讨论演示了功能依赖，当功能依赖不存在时 MySQL 产生的错误消息，以及在功能依赖不存在时导致 MySQL 接受查询的方法。

如果启用了 `ONLY_FULL_GROUP_BY`，则此查询可能无效，因为选择列表中的非聚合 `address` 列未在 `GROUP BY` 子句中命名：

```sql
SELECT name, address, MAX(age) FROM t GROUP BY name;
```

如果 `name` 是 `t` 的主键或是唯一的 `NOT NULL` 列，则查询是有效的。在这种情况下，MySQL 会认识到所选列在一个分组列上具有功能依赖关系。例如，如果 `name` 是主键，其值确定了 `address` 的值，因为每个组只有一个主键值，因此只有一行。因此，在组中选择 `address` 值时没有随机性，也不需要拒绝查询。

如果 `name` 不是 `t` 的主键或唯一的 `NOT NULL` 列，则查询是无效的。在这种情况下，无法推断出功能依赖关系，会发生错误：

```sql
mysql> SELECT name, address, MAX(age) FROM t GROUP BY name;
ERROR 1055 (42000): Expression #2 of SELECT list is not in GROUP
BY clause and contains nonaggregated column 'mydb.t.address' which
is not functionally dependent on columns in GROUP BY clause; this
is incompatible with sql_mode=only_full_group_by
```

如果你知道，*对于给定的数据集*，每个 `name` 值实际上唯一确定了 `address` 值，那么 `address` 实际上是依赖于 `name` 的。为了告诉 MySQL 接受这个查询，你可以使用 `ANY_VALUE()` 函数：

```sql
SELECT name, ANY_VALUE(address), MAX(age) FROM t GROUP BY name;
```

或者禁用 `ONLY_FULL_GROUP_BY`。

上面的例子相当简单。特别是，你不太可能仅对一个主键列进行分组，因为每个组只包含一行。要了解更复杂查询中的功能依赖的其他示例，请参见 第 14.19.4 节，“功能依赖的检测”。

如果一个查询有聚合函数但没有 `GROUP BY` 子句，则在启用 `ONLY_FULL_GROUP_BY` 的情况下，选择列表、`HAVING` 条件或 `ORDER BY` 列中不能有非聚合列：

```sql
mysql> SELECT name, MAX(age) FROM t;
ERROR 1140 (42000): In aggregated query without GROUP BY, expression
#1 of SELECT list contains nonaggregated column 'mydb.t.name'; this
is incompatible with sql_mode=only_full_group_by
```

没有 `GROUP BY`，只有一个组，对于选择哪个 `name` 值为该组是不确定的。在这种情况下，也可以使用 `ANY_VALUE()`，如果 MySQL 选择哪个 `name` 值并不重要：

```sql
SELECT ANY_VALUE(name), MAX(age) FROM t;
```

`ONLY_FULL_GROUP_BY` 也会影响使用 `DISTINCT` 和 `ORDER BY` 的查询处理。考虑一个包含三列 `c1`、`c2` 和 `c3` 的表 `t`，其中包含以下行：

```sql
c1 c2 c3
1  2  A
3  4  B
1  2  C
```

假设我们执行以下查询，期望结果按 `c3` 排序：

```sql
SELECT DISTINCT c1, c2 FROM t ORDER BY c3;
```

要对结果进行排序，必须先消除重复项。但是，在这样做时，我们应该保留第一行还是第三行？这种任意选择会影响 `c3` 的保留值，进而影响排序并使其变得任意。为了避免这个问题，如果任何一个 `ORDER BY` 表达式不满足以下条件，具有 `DISTINCT` 和 `ORDER BY` 的查询将被拒绝为无效：

+   该表达式等于选择列表中的一个

+   表达式引用的所有列并且属于查询选定的表的元素都在选择列表中

另一个 MySQL 对标准 SQL 的扩展允许在 `HAVING` 子句中引用选择列表中的别名表达式。例如，以下查询返回表 `orders` 中仅出现一次的 `name` 值：

```sql
SELECT name, COUNT(name) FROM orders
  GROUP BY name
  HAVING COUNT(name) = 1;
```

MySQL 扩展允许在聚合列的 `HAVING` 子句中使用别名：

```sql
SELECT name, COUNT(name) AS c FROM orders
  GROUP BY name
  HAVING c = 1;
```

标准 SQL 仅允许在 `GROUP BY` 子句中使用列表达式，因此像这样的语句是无效的，因为 `FLOOR(value/100)` 是一个非列表达式：

```sql
SELECT id, FLOOR(value/100)
  FROM *tbl_name*
  GROUP BY id, FLOOR(value/100);
```

MySQL 扩展了标准 SQL，允许在 `GROUP BY` 子句中使用非列表达式，并认为前述语句是有效的。

标准 SQL 也不允许在 `GROUP BY` 子句中使用别名。MySQL 扩展了标准 SQL，允许使用别名，因此编写查询的另一种方式如下：

```sql
SELECT id, FLOOR(value/100) AS val
  FROM *tbl_name*
  GROUP BY id, val;
```

别名 `val` 被视为 `GROUP BY` 子句中的列表达式。

在 `GROUP BY` 子句中存在非列表达式的情况下，MySQL 认可该表达式与选择列表中的表达式相等。这意味着启用 `ONLY_FULL_GROUP_BY` SQL 模式时，包含 `GROUP BY id, FLOOR(value/100)` 的查询是有效的，因为选择列表中也包含相同的 `FLOOR()` 表达式。然而，MySQL 不会尝试识别对 `GROUP BY` 非列表达式的函数依赖性，因此即使第三个选择的表达式是 `id` 列和 `GROUP BY` 子句中的 `FLOOR()` 表达式的简单公式，以下查询在启用 `ONLY_FULL_GROUP_BY` 的情况下是无效的：

```sql
SELECT id, FLOOR(value/100), id+FLOOR(value/100)
  FROM *tbl_name*
  GROUP BY id, FLOOR(value/100);
```

一个解决方法是使用派生表：

```sql
SELECT id, F, id+F
  FROM
    (SELECT id, FLOOR(value/100) AS F
     FROM *tbl_name*
     GROUP BY id, FLOOR(value/100)) AS dt;
```
