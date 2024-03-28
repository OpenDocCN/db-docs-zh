# 15.2.4 EXCEPT 子句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/except.html`](https://dev.mysql.com/doc/refman/8.0/en/except.html)

```sql
*query_expression_body* EXCEPT [ALL | DISTINCT] *query_expression_body*
    [EXCEPT [ALL | DISTINCT] *query_expression_body*]
    [...]

*query_expression_body*:
    *See Section 15.2.14, “Set Operations with UNION, INTERSECT, and EXCEPT”*
```

`EXCEPT`将第一个查询块的结果限制为那些在第二个查询块中（也）找不到的行。与`UNION`和`INTERSECT`一样，任一查询块都可以使用`SELECT`、`TABLE`或`VALUES`中的任何一个。下面是一个使用在第 15.2.8 节，“INTERSECT 子句”中定义的表`a`、`b`和`c`的示例：

```sql
mysql> TABLE a EXCEPT TABLE b;
+------+------+
| m    | n    |
+------+------+
|    2 |    3 |
+------+------+
1 row in set (0.00 sec)

mysql> TABLE a EXCEPT TABLE c;
+------+------+
| m    | n    |
+------+------+
|    1 |    2 |
|    2 |    3 |
+------+------+
2 rows in set (0.00 sec)

mysql> TABLE b EXCEPT TABLE c;
+------+------+
| m    | n    |
+------+------+
|    1 |    2 |
+------+------+
1 row in set (0.00 sec)
```

与`UNION`和`INTERSECT`一样，如果未指定`DISTINCT`或`ALL`，默认为`DISTINCT`。

`DISTINCT`会移除关系两侧发现的重复项，如下所示：

```sql
mysql> TABLE c EXCEPT DISTINCT TABLE a;
+------+------+
| m    | n    |
+------+------+
|    1 |    3 |
+------+------+
1 row in set (0.00 sec)

mysql> TABLE c EXCEPT ALL TABLE a;
+------+------+
| m    | n    |
+------+------+
|    1 |    3 |
|    1 |    3 |
+------+------+
2 rows in set (0.00 sec)
```

（第一个语句的效果与`TABLE c EXCEPT TABLE a`相同。）

与`UNION`或`INTERSECT`不同，`EXCEPT` *不* 是可交换的——也就是说，结果取决于操作数的顺序，如下所示：

```sql
mysql> TABLE a EXCEPT TABLE c;
+------+------+
| m    | n    |
+------+------+
|    1 |    2 |
|    2 |    3 |
+------+------+
2 rows in set (0.00 sec)

mysql> TABLE c EXCEPT TABLE a;
+------+------+
| m    | n    |
+------+------+
|    1 |    3 |
+------+------+
1 row in set (0.00 sec)
```

与`UNION`一样，要比较的结果集必须具有相同数量的列。结果集列类型也与`UNION`一样确定。

`EXCEPT`在 MySQL 8.0.31 中添加。
