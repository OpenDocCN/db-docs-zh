# 15.2.3 DO 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/do.html`](https://dev.mysql.com/doc/refman/8.0/en/do.html)

```sql
DO *expr* [, *expr*] ...
```

`DO`执行表达式但不返回任何结果。在大多数情况下，`DO`相当于`SELECT *`expr`*, ...`，但它的优势在于当你不关心结果时速度稍快。

`DO`主要用于具有副作用的函数，例如`RELEASE_LOCK()`。

示例：这个`SELECT`语句暂停，同时产生一个结果集：

```sql
mysql> SELECT SLEEP(5);
+----------+
| SLEEP(5) |
+----------+
|        0 |
+----------+
1 row in set (5.02 sec)
```

另一方面，`DO`暂停而不产生结果集。

```sql
mysql> DO SLEEP(5);
Query OK, 0 rows affected (4.99 sec)
```

例如，在禁止产生结果集的存储函数或触发器中，这可能很有用。

`DO`仅执行表达式。它不能在所有可以使用`SELECT`的情况下使用。例如，`DO id FROM t1`是无效的，因为它引用了一个表。
