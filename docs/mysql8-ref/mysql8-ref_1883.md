# 27.5.1 视图语法

> 原文：[`dev.mysql.com/doc/refman/8.0/en/view-syntax.html`](https://dev.mysql.com/doc/refman/8.0/en/view-syntax.html)

`CREATE VIEW`语句创建一个新视图（参见 Section 15.1.23, “CREATE VIEW Statement”）。要修改视图的定义或删除视图，请使用`ALTER VIEW`（参见 Section 15.1.11, “ALTER VIEW Statement”）或`DROP VIEW`（参见 Section 15.1.35, “DROP VIEW Statement”）。

视图可以从多种类型的`SELECT`语句创建。它可以引用基本表或其他视图。它可以使用连接、`UNION`和子查询。`SELECT`甚至不需要引用任何表。以下示例定义了一个视图，从另一个表中选择了两列，以及从这些列计算出的表达式：

```sql
mysql> CREATE TABLE t (qty INT, price INT);
mysql> INSERT INTO t VALUES(3, 50), (5, 60);
mysql> CREATE VIEW v AS SELECT qty, price, qty*price AS value FROM t;
mysql> SELECT * FROM v;
+------+-------+-------+
| qty  | price | value |
+------+-------+-------+
|    3 |    50 |   150 |
|    5 |    60 |   300 |
+------+-------+-------+
mysql> SELECT * FROM v WHERE qty = 5;
+------+-------+-------+
| qty  | price | value |
+------+-------+-------+
|    5 |    60 |   300 |
+------+-------+-------+
```
