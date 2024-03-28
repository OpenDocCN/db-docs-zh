> 原文：[`dev.mysql.com/doc/refman/8.0/en/correlated-subqueries.html`](https://dev.mysql.com/doc/refman/8.0/en/correlated-subqueries.html)

#### 15.2.15.7 相关子查询

*相关子查询* 是一个包含对外部查询中也出现的表的引用的子查询。例如：

```sql
SELECT * FROM t1
  WHERE column1 = ANY (SELECT column1 FROM t2
                       WHERE t2.column2 = t1.column2);
```

注意，子查询包含对 `t1` 列的引用，即使子查询的 `FROM` 子句没有提及表 `t1`。因此，MySQL 查找子查询外部，在外部查询中找到 `t1`。

假设表 `t1` 包含一行，其中 `column1 = 5` 和 `column2 = 6`；同时，表 `t2` 包含一行，其中 `column1 = 5` 和 `column2 = 7`。简单表达式 `... WHERE column1 = ANY (SELECT column1 FROM t2)` 将是 `TRUE`，但在这个例子中，子查询中的 `WHERE` 子句是 `FALSE`（因为 `(5,6)` 不等于 `(5,7)`），因此整个表达式是 `FALSE`。

**作用域规则：** MySQL 从内到外进行评估。例如：

```sql
SELECT column1 FROM t1 AS x
  WHERE x.column1 = (SELECT column1 FROM t2 AS x
    WHERE x.column1 = (SELECT column1 FROM t3
      WHERE x.column2 = t3.column1));
```

在这个语句中，`x.column2` 必须是表 `t2` 中的一列，因为 `SELECT column1 FROM t2 AS x ...` 重命名了 `t2`。它不是表 `t1` 中的一列，因为 `SELECT column1 FROM t1 ...` 是一个*更远处*的外部查询。

从 MySQL 8.0.24 开始，当 `optimizer_switch` 变量的 `subquery_to_derived` 标志启用时，优化器可以将相关标量子查询转换为派生表。考虑这里显示的查询：

```sql
SELECT * FROM t1 
    WHERE ( SELECT a FROM t2 
              WHERE t2.a=t1.a ) > 0;
```

为了避免为给定的派生表多次实例化，我们可以代替多次实例化一个派生表，该派生表在内部查询中引用的表（`t2.a`）上添加一个分组，然后在提升的谓词（`t1.a = derived.a`）上进行外连接，以选择正确的组与外部行匹配。 （如果子查询已经有明确的分组，则额外的分组将添加到分组列表的末尾。）因此，先前显示的查询可以像这样重写：

```sql
SELECT t1.* FROM t1 
    LEFT OUTER JOIN
        (SELECT a, COUNT(*) AS ct FROM t2 GROUP BY a) AS derived
    ON  t1.a = derived.a 
        AND 
        REJECT_IF(
            (ct > 1),
            "ERROR 1242 (21000): Subquery returns more than 1 row"
            )
    WHERE derived.a > 0;
```

在重写的查询中，`REJECT_IF()` 表示一个内部函数，用于测试给定条件（这里是比较 `ct > 1`）并在条件为真时引发给定错误（在本例中是 `ER_SUBQUERY_NO_1_ROW`）。这反映了优化器在评估 `JOIN` 或 `WHERE` 子句之前执行的基数检查，之后才评估任何提升的谓词，只有在子查询不返回多于一行时才执行。

只有满足以下条件时，才能执行这种类型的转换：

+   子查询可以是`SELECT`列表、`WHERE`条件或`HAVING`条件的一部分，但不能是`JOIN`条件的一部分，并且不能包含`LIMIT`或`OFFSET`子句。此外，子查询不能包含任何集合操作，如`UNION`。

+   `WHERE` 子句可以包含一个或多个谓词，并用`AND`组合。如果`WHERE` 子句包含一个`OR`子句，则无法进行转换。`WHERE` 子句中至少有一个谓词必须符合转换条件，且没有一个谓词可以拒绝转换。

+   要符合转换的条件，`WHERE` 子句谓词必须是一个等式谓词，其中每个操作数都应该是一个简单的列引用。没有其他谓词—包括其他比较谓词—符合转换条件。该谓词必须使用等号操作符`=`进行比较；在这种情况下，不支持空安全`≪=>`操作符。

+   只包含内部引用的`WHERE`子句谓词不符合转换条件，因为它可以在分组之前进行评估。只包含外部引用的`WHERE`子句谓词符合转换条件，即使它可以提升到外部查询块。这是通过在派生表中添加一个不带分组的基数检查来实现的。

+   要符合条件，`WHERE` 子句谓词必须有一个操作数仅包含内部引用，另一个操作数仅包含外部引用。如果由于此规则而使谓词不符合条件，则拒绝转换查询。

+   相关列只能存在于子查询的`WHERE`子句中（而不是`SELECT`列表、`JOIN`或`ORDER BY`子句、`GROUP BY`列表或`HAVING`子句）。子查询的`FROM`列表中也不能有任何相关列。

+   相关列不能包含在聚合函数的参数列表中。

+   相关列必须在直接包含待转换子查询的查询块中解析。

+   在`WHERE`子句中的嵌套标量子查询中不能存在相关列。

+   子查询不能包含任何窗口函数，并且不能包含在子查询外部的查询块中聚合的任何聚合函数。如果`SELECT`列表元素中包含`COUNT()`聚合函数，则必须在最高级别，并且不能是表达式的一部分。

另请参阅第 15.2.15.8 节，“派生表”。
