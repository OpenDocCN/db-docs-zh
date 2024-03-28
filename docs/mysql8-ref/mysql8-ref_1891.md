# 27.9 视图的限制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/view-restrictions.html`](https://dev.mysql.com/doc/refman/8.0/en/view-restrictions.html)

在视图定义中引用的表的最大数量为 61。

视图处理未经优化：

+   不可能在视图上创建索引。

+   使用合并算法处理的视图可以使用索引。然而，使用 temptable 算法处理的视图无法利用其基础表的索引（尽管在生成临时表时可以使用索引）。

有一个普遍原则，即您不能在子查询中修改表并从同一表中进行选择。参见 Section 15.2.15.12, “子查询的限制”。

如果您从选择表的视图中选择，如果视图从子查询中选择表，并且使用合并算法评估视图，则也适用相同原则。例如：

```sql
CREATE VIEW v1 AS
SELECT * FROM t2 WHERE EXISTS (SELECT 1 FROM t1 WHERE t1.a = t2.a);

UPDATE t1, v2 SET t1.a = 1 WHERE t1.b = v2.b;
```

如果视图使用临时表进行评估，您*可以*从视图子查询中选择表，并在外部查询中修改该表。在这种情况下，视图存储在临时表中，因此您实际上并没有同时从子查询中选择表并修改它。（这是您可能希望通过在视图定义中指定 `ALGORITHM = TEMPTABLE` 强制 MySQL 使用 temptable 算法的另一个原因。）

您可以使用 `DROP TABLE` 或 `ALTER TABLE` 删除或更改视图定义中使用的表。即使这使视图无效，`DROP` 或 `ALTER` 操作也不会产生警告。相反，在使用视图时会稍后出现错误。可以使用 `CHECK TABLE` 来检查已被 `DROP` 或 `ALTER` 操作使无效的视图。

关于视图的可更新性，视图的总体目标是，如果任何视图在理论上是可更新的，那么在实践中它应该是可更新的。许多在理论上可更新的视图现在可以更新，但仍然存在限制。有关详细信息，请参见 Section 27.5.3, “可更新和可插入的视图”。

当前视图实现存在一个缺陷。如果用户被授予创建视图所需的基本权限（`CREATE VIEW` 和 `SELECT` 权限），那么该用户除非也被授予 `SHOW VIEW` 权限，否则不能调用 `SHOW CREATE VIEW` 查看该对象。

这个缺陷可能导致使用**mysqldump**备份数据库时出现问题，可能由于权限不足而失败。这个问题在 Bug #22062 中有描述。

解决这个问题的方法是管理员手动授予`SHOW VIEW`权限给被授予`CREATE VIEW`权限的用户，因为 MySQL 在创建视图时不会隐式授予该权限。

视图没有索引，因此索引提示不适用。在从视图中进行选择时，不允许使用索引提示。

`SHOW CREATE VIEW`使用`AS *alias_name*`子句显示视图定义中的每个列。如果列是从表达式创建的，则默认别名是表达式文本，可能会很长。在`CREATE VIEW`语句中，列名的别名会被检查是否超过 64 个字符的最大列长度（而不是 256 个字符的最大别名长度）。因此，如果任何列别名超过 64 个字符，则从`SHOW CREATE VIEW`输出创建的视图会失败。这可能会导致以下情况的问题，对于具有过长别名的视图：

+   视图定义无法复制到强制执行列长度限制的新副本中。

+   使用**mysqldump**创建的转储文件无法加载到强制执行列长度限制的服务器中。

为了解决这两个问题，一个解决方法是修改每个有问题的视图定义，使用提供更短列名的别名。然后视图就能正确复制，并且可以在不引起错误的情况下进行转储和重新加载。要修改定义，可以使用`DROP VIEW`和`CREATE VIEW`重新创建视图，或者用`CREATE OR REPLACE VIEW`替换定义。

对于在转储文件中重新加载视图定义时出现的问题，另一个解决方法是编辑转储文件以修改其`CREATE VIEW`语句。然而，这并不会改变原始视图定义，这可能会导致后续转储操作出现问题。