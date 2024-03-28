> 原文：[`dev.mysql.com/doc/refman/8.0/en/semijoins.html`](https://dev.mysql.com/doc/refman/8.0/en/semijoins.html)。

#### 10.2.2.1 使用半连接转换优化 IN 和 EXISTS 子查询谓词

半连接是一种准备时间转换，可以启用多种执行策略，如表拉出、重复消除、首次匹配、宽松扫描和物化。优化器使用半连接策略来改善子查询执行，如本节所述。

对于两个表之间的内连接，连接将从一个表中返回一行，只要在另一个表中有匹配。但对于一些问题，唯一重要的信息是是否有匹配，而不是匹配的次数。假设有名为`class`和`roster`的表，分别列出课程课程和班级名单（每个班级中注册的学生），要列出实际有学生注册的班级，可以使用这个连接：

```sql
SELECT class.class_num, class.class_name
    FROM class
    INNER JOIN roster
    WHERE class.class_num = roster.class_num;
```

然而，结果为每个注册学生列出每个班级一次。对于所提出的问题，这是信息的不必要重复。

假设`class`表中的`class_num`是主键，通过使用`SELECT DISTINCT`可以实现去重，但是先生成所有匹配行再后来消除重复是低效的。

可以通过使用子查询获得相同的无重复结果：

```sql
SELECT class_num, class_name
    FROM class
    WHERE class_num IN
        (SELECT class_num FROM roster);
```

这里，优化器可以识别到`IN`子句要求子查询只返回`roster`表中每个班级编号的一个实例。在这种情况下，查询可以使用半连接；也就是说，只返回与`roster`中的行匹配的`class`中每行的一个实例。

包含`EXISTS`子查询谓词的以下语句与包含等效`IN`子查询谓词的先前语句等效：

```sql
SELECT class_num, class_name
    FROM class
    WHERE EXISTS
        (SELECT * FROM roster WHERE class.class_num = roster.class_num);
```

在 MySQL 8.0.16 及更高版本中，任何带有`EXISTS`子查询谓词的语句都会受到与具有等效`IN`子查询谓词的语句相同的半连接转换的影响。

从 MySQL 8.0.17 开始，以下子查询被转换为反连接：

+   `NOT IN (SELECT ... FROM ...)`。

+   `NOT EXISTS (SELECT ... FROM ...)`。

+   `IN (SELECT ... FROM ...) IS NOT TRUE`。

+   `EXISTS (SELECT ... FROM ...) IS NOT TRUE`。

+   `IN (SELECT ... FROM ...) IS FALSE`。

+   `EXISTS (SELECT ... FROM ...) IS FALSE`。

简而言之，对形式为`IN (SELECT ... FROM ...)`或`EXISTS (SELECT ... FROM ...)`的子查询的否定被转换为反连接。

反连接是一种仅返回没有匹配的行的操作。考虑这里显示的查询：

```sql
SELECT class_num, class_name
    FROM class
    WHERE class_num NOT IN
        (SELECT class_num FROM roster);
```

此查询在内部重写为反连接`SELECT class_num, class_name FROM class ANTIJOIN roster ON class_num`，它返回`class`中每行的一个实例，该实例*不*与`roster`中的任何行匹配。这意味着对于`class`中的每行，一旦在`roster`中找到匹配，就可以丢弃`class`中的行。

如果被比较的表达式是可空的，则通常无法应用反连接转换。一个例外是 `(... NOT IN (SELECT ...)) IS NOT FALSE` 及其等效的 `(... IN (SELECT ...)) IS NOT TRUE` 可以被转换为反连接。

外连接和内连接语法允许在外部查询规范中，表引用可以是基本表、派生表、视图引用或公共表达式。

在 MySQL 中，一个子查询必须满足这些条件才能被处理为半连接（或在 MySQL 8.0.17 及更高版本中，如果 `NOT` 修改子查询，则为反连接）：

+   它必须是 `WHERE` 或 `ON` 子句的顶层出现的 `IN`、`= ANY` 或 `EXISTS` 谓词的一部分，可能作为 `AND` 表达式中的一个项。例如：

    ```sql
    SELECT ...
        FROM ot1, ...
        WHERE (oe1, ...) IN
            (SELECT ie1, ... FROM it1, ... WHERE ...);
    ```

    这里，`ot_*`i`*` 和 `it_*`i`*` 代表查询的外部和内部部分中的表，`oe_*`i`*` 和 `ie_*`i`*` 代表引用外部和内部表中列的表达式。

    在 MySQL 8.0.17 及更高版本中，子查询也可以作为被 `NOT`、`IS [NOT] TRUE` 或 `IS [NOT] FALSE` 修饰的表达式的参数。

+   它必须是一个单独的 `SELECT`，不包含 `UNION` 结构。

+   它不能包含 `HAVING` 子句。

+   它不能包含任何聚合函数（无论是显式还是隐式分组）。

+   它不能有 `LIMIT` 子句。

+   语句不能在外部查询中使用 `STRAIGHT_JOIN` 连接类型。

+   `STRAIGHT_JOIN` 修饰符不能存在。

+   外部和内部表的数量总和必须小于联接中允许的最大表数量。

+   子查询可以是相关的或不相关的。在 MySQL 8.0.16 及更高版本中，解相关性会查看作为 `EXISTS` 参数使用的子查询的 `WHERE` 子句中的平凡相关谓词，并使其能够优化，就像它是在 `IN (SELECT b FROM ...)` 中使用一样。术语 *平凡相关* 意味着谓词是一个相等谓词，它是 `WHERE` 子句中唯一的谓词（或与 `AND` 结合），并且一个操作数来自子查询中引用的表，另一个操作数来自外部查询块。

+   `DISTINCT` 关键字是允许的，但会被忽略。半连接策略会自动处理重复项的移除。

+   允许使用 `GROUP BY` 子句，但会被忽略，除非子查询还包含一个或多个聚合函数。

+   允许使用 `ORDER BY` 子句，但会被忽略，因为排序对于半连接策略的评估是无关紧要的。

如果一个子查询符合上述条件，MySQL 将其转换为半连接（或在 MySQL 8.0.17 或更高版本中，如果适用，转换为反连接）并从以下策略中做出基于成本的选择：

+   将子查询转换为联接，或使用表拉出并在子查询表和外部表之间运行查询作为内连接。表拉出是将一个表从子查询中拉出到外部查询中。

+   *Duplicate Weedout*: 将半连接视为连接运行，并使用临时表去除重复记录。

+   *FirstMatch*: 在扫描内部表以获取行组合时，如果给定值组有多个实例，则选择一个而不是返回它们全部。这种“捷径”扫描并消除了不必要的行的生成。

+   *LooseScan*: 使用允许从每个子查询值组中选择单个值的索引扫描子查询表。

+   将子查询材料化为用于执行连接的索引临时表，其中索引用于去重。当将临时表与外部表连接时，索引也可能稍后用于查找；如果不是，则扫描表。有关材料化的更多信息，请参见第 10.2.2.2 节，“使用材料化优化子查询”。

可以使用以下`optimizer_switch`系统变量标志启用或禁用这些策略：

+   `semijoin`标志控制是否使用半连接。从 MySQL 8.0.17 开始，这也适用于反连接。

+   如果启用了`semijoin`，则`firstmatch`、`loosescan`、`duplicateweedout`和`materialization`标志可以更精细地控制允许的半连接策略。

+   如果禁用了`duplicateweedout`半连接策略，则除非所有其他适用策略也被禁用，否则不会使用。

+   如果禁用了`duplicateweedout`，优化器偶尔可能生成远非最佳的查询计划。这是由于贪婪搜索期间的启发式修剪导致的，可以通过设置`optimizer_prune_level=0`来避免。

这些标志默认启用。请参见第 10.9.2 节，“可切换优化”。

优化器最小化了对视图和派生表处理的差异。这影响了使用`STRAIGHT_JOIN`修饰符和具有可以转换为半连接的`IN`子查询的视图的查询。以下查询说明了这一点，因为处理方式的改变导致了转换的改变，从而产生了不同的执行策略：

```sql
CREATE VIEW v AS
SELECT *
FROM t1
WHERE a IN (SELECT b
           FROM t2);

SELECT STRAIGHT_JOIN *
FROM t3 JOIN v ON t3.x = v.a;
```

优化器首先查看视图并将 `IN` 子查询转换为半连接，然后检查是否可能将视图合并到外部查询中。由于外部查询中的 `STRAIGHT_JOIN` 修饰符阻止了半连接，优化器拒绝合并，导致使用物化表进行派生表评估。

`EXPLAIN` 输出指示了半连接策略的使用如下：

+   对于扩展的 `EXPLAIN` 输出，随后由 `SHOW WARNINGS` 显示的文本显示了重写的查询，其中显示了半连接结构。从中可以了解哪些表被提取出半连接。如果子查询被转换为半连接，您应该看到子查询谓词消失，其表和 `WHERE` 子句被合并到外部查询的连接列表和 `WHERE` 子句中。

+   在 `Extra` 列中的 `Start temporary` 和 `End temporary` 表示重复消除时使用了临时表。未被提取出来且在 `Start temporary` 和 `End temporary` 覆盖的 `EXPLAIN` 输出行范围内的表，在临时表中具有其 `rowid`。

+   `FirstMatch(*`tbl_name`*)` 在 `Extra` 列中表示连接快捷方式。

+   `LooseScan(*`m`*..*`n`*)` 在 `Extra` 列中表示使用了 LooseScan 策略。*`m`* 和 *`n`* 是关键部件编号。

+   临时表用于物化的使用通过具有 `select_type` 值为 `MATERIALIZED` 和具有 `table` 值为 `<subquery*`N`*>` 的行来表示。

在 MySQL 8.0.21 及更高版本中，半连接转换也可以应用于使用 `[NOT] IN` 或 `[NOT] EXISTS` 子查询谓词的单表 `UPDATE` 或 `DELETE` 语句，前提是该语句不使用 `ORDER BY` 或 `LIMIT`，并且通过优化器提示或 `optimizer_switch` 设置允许半连接转换。
