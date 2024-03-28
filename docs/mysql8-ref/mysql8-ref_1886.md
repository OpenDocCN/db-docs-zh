# 27.5.4 视图 WITH CHECK OPTION 子句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/view-check-option.html`](https://dev.mysql.com/doc/refman/8.0/en/view-check-option.html)

可以为可更新视图提供`WITH CHECK OPTION`子句，以防止插入到`select_statement`中`WHERE`子句不为真的行。它还防止更新使`WHERE`子句为真但更新会导致其不为真的行（换句话说，它防止可见行被更新为不可见行）。

在可更新视图的`WITH CHECK OPTION`子句中，`LOCAL`和`CASCADED`关键字确定了在视图以另一个视图的形式定义时进行检查测试的范围。当没有给出关键字时，默认为`CASCADED`。

`WITH CHECK OPTION`测试符合标准：

+   使用`LOCAL`，视图`WHERE`子句会被检查，然后检查递归到底层视图并应用相同的规则。

+   使用`CASCADED`，视图`WHERE`子句会被检查，然后检查递归到底层视图，为它们添加`WITH CASCADED CHECK OPTION`（用于检查目的；它们的定义保持不变），并应用相同的规则。

+   没有检查选项时，视图`WHERE`子句不会被检查，然后检查会递归到底层视图，并应用相同的规则。

考虑以下表和一组视图的定义：

```sql
CREATE TABLE t1 (a INT);
CREATE VIEW v1 AS SELECT * FROM t1 WHERE a < 2
WITH CHECK OPTION;
CREATE VIEW v2 AS SELECT * FROM v1 WHERE a > 0
WITH LOCAL CHECK OPTION;
CREATE VIEW v3 AS SELECT * FROM v1 WHERE a > 0
WITH CASCADED CHECK OPTION;
```

这里的`v2`和`v3`视图是以另一个视图`v1`的形式定义的。

对于`v2`的插入会根据其`LOCAL`检查选项进行检查，然后检查递归到`v1`并再次应用规则。`v1`的规则导致检查失败。`v3`的检查也失败：

```sql
mysql> INSERT INTO v2 VALUES (2);
ERROR 1369 (HY000): CHECK OPTION failed 'test.v2'
mysql> INSERT INTO v3 VALUES (2);
ERROR 1369 (HY000): CHECK OPTION failed 'test.v3'
```
