> 原文：[`dev.mysql.com/doc/refman/8.0/en/subquery-materialization.html`](https://dev.mysql.com/doc/refman/8.0/en/subquery-materialization.html)

#### 10.2.2.2 使用物化优化子查询

优化器使用物化来实现更高效的子查询处理。物化通过生成一个子查询结果作为临时表来加快查询执行，通常在内存中。当 MySQL 首次需要子查询结果时，它将该结果物化为临时表。任何后续需要结果的时候，MySQL 再次引用临时表。优化器可以使用哈希索引对表进行索引，以使查找快速且廉价。索引包含唯一值以消除重复项并使表变得更小。

当可能时，子查询物化使用内存临时表，如果表变得太大，则回退到磁盘存储。参见第 10.4.4 节，“MySQL 中的内部临时表使用”。

如果不使用物化，优化器有时会将非相关子查询重写为相关子查询。例如，以下`IN`子查询是非相关的（*`where_condition`*仅涉及`t2`列而不涉及`t1`列）：

```sql
SELECT * FROM t1
WHERE t1.a IN (SELECT t2.b FROM t2 WHERE *where_condition*);
```

优化器可能会将此重写为`EXISTS`相关子查询：

```sql
SELECT * FROM t1
WHERE EXISTS (SELECT t2.b FROM t2 WHERE *where_condition* AND t1.a=t2.b);
```

使用临时表进行子查询物化可以避免这种重写，并使得可以仅执行一次子查询，而不是对外部查询的每一行执行一次。

要在 MySQL 中使用子查询物化，必须启用`optimizer_switch`系统变量`materialization`标志。（参见第 10.9.2 节，“可切换的优化”。）启用`materialization`标志后，物化适用于出现在任何地方的子查询谓词（在选择列表中，`WHERE`，`ON`，`GROUP BY`，`HAVING`或`ORDER BY`），适用于以下任何用例的谓词：

+   当外部表达式*`oe_i`*或内部表达式*`ie_i`*不可为空时，谓词具有此形式。*`N`*为 1 或更大。

    ```sql
    (*oe_1*, *oe_2*, ..., *oe_N*) [NOT] IN (SELECT *ie_1*, *i_2*, ..., *ie_N* ...)
    ```

+   当存在单个外部表达式*`oe`*和内部表达式*`ie`*时，谓词具有此形式。这些表达式可以为空。

    ```sql
    *oe* [NOT] IN (SELECT *ie* ...)
    ```

+   谓词为`IN`或`NOT IN`，并且`UNKNOWN`（`NULL`）的结果与`FALSE`的结果具有相同的含义。

以下示例说明了`UNKNOWN`和`FALSE`谓词评估等价性要求如何影响是否可以使用子查询物化。假设*`where_condition`*仅涉及`t2`列而不涉及`t1`列，因此子查询是非相关的。

此查询需要进行物化：

```sql
SELECT * FROM t1
WHERE t1.a IN (SELECT t2.b FROM t2 WHERE *where_condition*);
```

在这里，`IN` 谓词返回 `UNKNOWN` 或 `FALSE` 都无关紧要。无论如何，来自 `t1` 的行都不会包含在查询结果中。

一个不使用子查询物化的示例是下面的查询，其中 `t2.b` 是一个可空列：

```sql
SELECT * FROM t1
WHERE (t1.a,t1.b) NOT IN (SELECT t2.a,t2.b FROM t2
                          WHERE *where_condition*);
```

使用子查询物化有以下限制：

+   内部和外部表达式的类型必须匹配。例如，如果两个表达式都是整数或都是十进制数，优化器可能可以使用物化，但如果一个表达式是整数，另一个是十进制数，则不能使用物化。

+   内部表达式不能是一个 `BLOB`。

使用带有查询的 `EXPLAIN` 提供了一些指示，表明优化器是否使用了子查询物化：

+   与不使用物化的查询执行相比，`select_type` 可能会从 `DEPENDENT SUBQUERY` 变为 `SUBQUERY`。这表明，对于每个外部行执行一次的子查询，物化使得子查询只需执行一次。

+   对于扩展的 `EXPLAIN` 输出，后续 `SHOW WARNINGS` 显示的文本包括 `materialize` 和 `materialized-subquery`。

在 MySQL 8.0.21 及更高版本中，MySQL 还可以对使用 `[NOT] IN` 或 `[NOT] EXISTS` 子查询谓词的单表 `UPDATE` 或 `DELETE` 语句应用子查询物化，前提是该语句不使用 `ORDER BY` 或 `LIMIT`，并且优化器提示或 `optimizer_switch` 设置允许使用子查询物化。
