> 原文：[`dev.mysql.com/doc/refman/8.0/en/outer-join-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/outer-join-optimization.html)

#### 10.2.1.9 外连接优化

外连接包括`LEFT JOIN`和`RIGHT JOIN`。

MySQL 实现`*`A`* LEFT JOIN *`B`* *`join_specification`*`如下：

+   表*`B`*被设置为依赖于表*`A`*和*`A`*依赖的所有表。

+   表*`A`*被设置为依赖于所有表（除了*`B`*）在`LEFT JOIN`条件中使用的表。

+   `LEFT JOIN`条件用于决定如何从表*`B`*中检索行。（换句话说，`WHERE`子句中的任何条件都不会被使用。）

+   执行所有标准连接优化，除了一个表始终在其依赖的所有表之后读取。如果存在循环依赖关系，则会出现错误。

+   执行所有标准`WHERE`优化。

+   如果*`A`*中有一行与`WHERE`子句匹配，但*`B`*中没有一行与`ON`条件匹配，则会生成一个额外的*`B`*行，其中所有列均设置为`NULL`。

+   如果您使用`LEFT JOIN`查找在某个表中不存在的行，并且在`WHERE`部分中有以下测试：`*`col_name`* IS NULL`，其中*`col_name`*被声明为`NOT NULL`的列，那么 MySQL 在找到与`LEFT JOIN`条件匹配的一行后，停止搜索更多行（对于特定键组合）。

`RIGHT JOIN`的实现类似于`LEFT JOIN`，只是表的角色被颠倒。右连接被转换为等效的左连接，如 Section 10.2.1.10, “Outer Join Simplification”中所述。

对于`LEFT JOIN`，如果生成的`NULL`行的`WHERE`条件始终为假，则`LEFT JOIN`会转为内连接。例如，在以下查询中，如果`t2.column1`为`NULL`，则`WHERE`子句将为假：

```sql
SELECT * FROM t1 LEFT JOIN t2 ON (column1) WHERE t2.column2=5;
```

因此，将查询转换为内连接是安全的：

```sql
SELECT * FROM t1, t2 WHERE t2.column2=5 AND t1.column1=t2.column1;
```

在 MySQL 8.0.14 及更高版本中，常量文字表达式导致的平凡`WHERE`条件在准备期间被移除，而不是在优化的后期阶段，此时连接已经被简化。提前移除平凡条件允许优化器将外连接转换为内连接；这可以改善包含`WHERE`子句中包含平凡条件的外连接查询的计划，例如以下查询：

```sql
SELECT * FROM t1 LEFT JOIN t2 ON *condition_1* WHERE *condition_2* OR 0 = 1
```

优化器现在在准备期间看到 0 = 1 始终为假，使得`OR 0 = 1`多余，并将其删除，留下这样的内容：

```sql
SELECT * FROM t1 LEFT JOIN t2 ON *condition_1* where *condition_2*
```

现在优化器可以将查询重写为内连接，如下所示：

```sql
SELECT * FROM t1 JOIN t2 WHERE *condition_1* AND *condition_2*
```

现在优化器可以在表`t1`之前使用表`t2`，如果这样做可以得到更好的查询计划。要提供关于表连接顺序的提示，请使用优化器提示；参见第 10.9.3 节，“优化器提示”。或者，使用`STRAIGHT_JOIN`；参见第 15.2.13 节，“SELECT 语句”。然而，`STRAIGHT_JOIN`可能会阻止索引的使用，因为它禁用了半连接转换；参见第 10.2.2.1 节，“使用半连接转换优化 IN 和 EXISTS 子查询谓词”。
