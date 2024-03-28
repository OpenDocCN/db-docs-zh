# 27.5.2 视图处理算法

> 原文：[`dev.mysql.com/doc/refman/8.0/en/view-algorithms.html`](https://dev.mysql.com/doc/refman/8.0/en/view-algorithms.html)

`CREATE VIEW` 或 `ALTER VIEW` 的可选 `ALGORITHM` 子句是 MySQL 对标准 SQL 的扩展。它影响 MySQL 处理视图的方式。`ALGORITHM` 有三个值：`MERGE`、`TEMPTABLE` 或 `UNDEFINED`。

+   对于 `MERGE`，引用视图的语句文本和视图定义被合并，以便视图定义的部分替换语句的相应部分。

+   对于 `TEMPTABLE`，视图的结果被检索到一个临时表中，然后用于执行语句。

+   对于 `UNDEFINED`，MySQL 会选择使用哪种算法。如果可能的话，它会优先选择 `MERGE` 而不是 `TEMPTABLE`，因为 `MERGE` 通常更有效率，而且如果使用临时表，则视图无法更新。

+   如果没有 `ALGORITHM` 子句，则默认算法由 `optimizer_switch` 系统变量的 `derived_merge` 标志的值确定。有关更多讨论，请参见 Section 10.2.2.4, “Optimizing Derived Tables, View References, and Common Table Expressions with Merging or Materialization”。

明确指定 `TEMPTABLE` 的一个原因是，在创建临时表后并在用于完成语句处理之前，可以释放对基础表的锁。这可能导致比 `MERGE` 算法更快地释放锁，以便使用视图的其他客户端不会被阻塞太久。

视图算法可能为 `UNDEFINED` 有三个原因：

+   `CREATE VIEW` 语句中没有 `ALGORITHM` 子句。

+   `CREATE VIEW` 语句具有显式的 `ALGORITHM = UNDEFINED` 子句。

+   为只能使用临时表处理的视图指定了 `ALGORITHM = MERGE`。在这种情况下，MySQL 会生成警告并将算法设置为 `UNDEFINED`。

如前所述，`MERGE` 是通过将视图定义的相应部分合并到引用该视图的语句中来处理的。以下示例简要说明了 `MERGE` 算法的工作原理。这些示例假设存在一个具有以下定义的视图 `v_merge`：

```sql
CREATE ALGORITHM = MERGE VIEW v_merge (vc1, vc2) AS
SELECT c1, c2 FROM t WHERE c3 > 100;
```

示例 1：假设我们发出以下语句：

```sql
SELECT * FROM v_merge;
```

MySQL 处理该语句如下：

+   `v_merge` 变为 `t`

+   `*` 变为 `vc1, vc2`，对应于 `c1, c2`

+   视图 `WHERE` 子句被添加

要执行的结果语句变为：

```sql
SELECT c1, c2 FROM t WHERE c3 > 100;
```

示例 2：假设我们发出以下语句：

```sql
SELECT * FROM v_merge WHERE vc1 < 100;
```

这个语句的处理方式与前一个类似，只是`vc1 < 100`变成了`c1 < 100`，并且视图的`WHERE`子句被添加到语句的`WHERE`子句中，使用`AND`连接词（并且添加括号以确保子句的部分以正确的优先级执行）。最终要执行的语句变为：

```sql
SELECT c1, c2 FROM t WHERE (c3 > 100) AND (c1 < 100);
```

实际上，要执行的语句具有以下形式的`WHERE`子句：

```sql
WHERE (select WHERE) AND (view WHERE)
```

如果无法使用`MERGE`算法，则必须使用临时表。阻止合并的结构与阻止派生表和公共表达式合并的结构相同。例如，在子查询中使用`SELECT DISTINCT`或`LIMIT`。有关详细信息，请参见 Section 10.2.2.4，“使用合并或实体化优化派生表、视图引用和公共表达式”。
