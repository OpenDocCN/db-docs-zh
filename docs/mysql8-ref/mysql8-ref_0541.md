> 原文：[`dev.mysql.com/doc/refman/8.0/en/derived-table-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/derived-table-optimization.html)

#### 10.2.2.4 优化派生表、视图引用和公共表达式的合并或物化

优化器可以使用两种策略处理派生表引用（也适用于视图引用和公共表达式）：

+   将派生表合并到外部查询块中

+   将派生表物化为内部临时表

示例 1：

```sql
SELECT * FROM (SELECT * FROM t1) AS derived_t1;
```

通过将派生表`derived_t1`合并，该查询的执行方式类似于：

```sql
SELECT * FROM t1;
```

示例 2：

```sql
SELECT *
  FROM t1 JOIN (SELECT t2.f1 FROM t2) AS derived_t2 ON t1.f2=derived_t2.f1
  WHERE t1.f1 > 0;
```

通过将派生表`derived_t2`合并，该查询的执行方式类似于：

```sql
SELECT t1.*, t2.f1
  FROM t1 JOIN t2 ON t1.f2=t2.f1
  WHERE t1.f1 > 0;
```

通过物化，`derived_t1`和`derived_t2`在各自的查询中被视为单独的表。

优化器处理派生表、视图引用和公共表达式的方式相同：尽可能避免不必要的物化，这使得可以将外部查询的条件下推到派生表，并生成更有效的执行计划。（例如，参见 Section 10.2.2.2, “Optimizing Subqueries with Materialization”。）

如果合并会导致外部查询块引用超过 61 个基本表，则优化器选择物化。

如果这些条件都成立，优化器将派生表或视图引用中的`ORDER BY`子句传播到外部查询块：

+   外部查询没有分组或聚合。

+   外部查询没有指定`DISTINCT`、`HAVING`或`ORDER BY`。

+   外部查询在`FROM`子句中只有这个派生表或视图引用作为唯一数据源。

否则，优化器将忽略`ORDER BY`子句。

可以影响优化器是否尝试将派生表、视图引用和公共表达式合并到外部查询块的以下方法：

+   可以使用`MERGE`和`NO_MERGE`优化器提示。假设没有其他规则阻止合并，则应用这些提示。参见 Section 10.9.3, “Optimizer Hints”。

+   同样，您可以使用`optimizer_switch`系统变量的`derived_merge`标志。默认情况下，该标志已启用以允许合并。禁用该标志将阻止合并并避免`ER_UPDATE_TABLE_USED`错误。

    `derived_merge`标志也适用于不包含`ALGORITHM`子句的视图。因此，如果对使用等效于子查询的表达式的视图引用发生`ER_UPDATE_TABLE_USED`错误，则将`ALGORITHM=TEMPTABLE`添加到视图定义中可以阻止合并，并优先于`derived_merge`值。

+   通过在子查询中使用任何阻止合并的构造，可以禁用合并，尽管这些构造对于材料化的影响不够明显。阻止合并的构造对于派生表、公共表达式和视图引用是相同的：

    +   聚合函数或窗口函数（`SUM()`、`MIN()`、`MAX()`、`COUNT()`等）

    +   `DISTINCT`

    +   `GROUP BY`

    +   `HAVING`

    +   `LIMIT`

    +   `UNION`或`UNION ALL`

    +   在选择列表中的子查询

    +   对用户变量的赋值

    +   仅引用文字值（在这种情况下，没有基础表）

如果优化器选择材料化策略而不是合并派生表，则处理查询如下：

+   优化器推迟派生表的材料化，直到在查询执行期间需要其内容。这样做可以提高性能，因为延迟材料化可能导致根本不需要进行材料化。考虑一个将派生表的结果与另一个表连接的查询：如果优化器首先处理另一个表并发现它不返回任何行，则无需继续执行连接，并且优化器可以完全跳过材料化派生表。

+   在查询执行期间，优化器可能向派生表添加索引，以加快从中检索行的速度。

考虑以下包含派生表的`EXPLAIN`语句，用于一个包含派生表的`SELECT`查询：

```sql
EXPLAIN SELECT * FROM (SELECT * FROM t1) AS derived_t1;
```

优化器通过延迟派生表的材料化直到在`SELECT`执行期间需要结果时来避免材料化派生表。在这种情况下，查询不会被执行（因为它出现在`EXPLAIN`语句中），因此结果永远不会被需要。

即使对于执行的查询，延迟派生表的材料化也可能使优化器完全避免材料化。当发生这种情况时，查询执行速度比执行材料化所需的时间更快。考虑以下查询，该查询将派生表的结果与另一个表连接：

```sql
SELECT *
  FROM t1 JOIN (SELECT t2.f1 FROM t2) AS derived_t2
          ON t1.f2=derived_t2.f1
  WHERE t1.f1 > 0;
```

如果优化过程首先处理`t1`，并且`WHERE`子句产生空结果，则连接必然为空，衍生表不需要实体化。

对于需要实体化的衍生表，优化器可能会向实体化表添加索引以加快对其的访问速度。如果此类索引使得可以使用`ref`访问表，则可以大大减少查询执行期间读取的数据量。考虑以下查询：

```sql
SELECT *
 FROM t1 JOIN (SELECT DISTINCT f1 FROM t2) AS derived_t2
         ON t1.f1=derived_t2.f1;
```

如果构建索引`derived_t2`中的列`f1`可以使最低成本执行计划使用`ref`访问，优化器会自动添加索引。添加索引后，优化器可以将实体化的衍生表视为具有索引的常规表，并且从生成的索引中获得类似的好处。与没有索引的查询执行成本相比，索引创建的开销微不足道。如果`ref`访问的成本高于其他访问方法，优化器不会创建索引，也不会有任何损失。

对于优化器跟踪输出，合并的衍生表或视图引用不会显示为节点。只有其基础表会出现在顶层查询计划中。

衍生表的实体化适用于通用表达式（CTEs）。此外，以下考虑事项特别适用于 CTEs。

如果查询通过查询实体化了一个 CTE，则即使查询多次引用它，也只会为查询实体化一次。

递归 CTE 始终会被实体化。

如果 CTE 被实体化，优化器会自动添加相关索引，如果估计索引可以加快顶层语句对 CTE 的访问速度。这类似于自动为衍生表创建索引，不同之处在于，如果 CTE 被多次引用，优化器可能会创建多个索引，以最适当的方式加快每个引用的访问速度。

`MERGE`和`NO_MERGE`优化提示可以应用于 CTEs。顶层语句中的每个 CTE 引用都可以有自己的提示，允许选择性地合并或实体化 CTE 引用。以下语句使用提示指示`cte1`应该合并，`cte2`应该实体化：

```sql
WITH
  cte1 AS (SELECT a, b FROM table1),
  cte2 AS (SELECT c, d FROM table2)
SELECT /*+ MERGE(cte1) NO_MERGE(cte2) */ cte1.b, cte2.d
FROM cte1 JOIN cte2
WHERE cte1.a = cte2.c;
```

`CREATE VIEW`的`ALGORITHM`子句不会影响视图定义中的`SELECT`语句之前的任何`WITH`")子句的实体化。考虑以下语句：

```sql
CREATE ALGORITHM={TEMPTABLE|MERGE} VIEW v1 AS WITH ... SELECT ...
```

`ALGORITHM`值仅影响`SELECT`的实体化，而不影响`WITH`")子句。

在 MySQL 8.0.16 之前，如果`internal_tmp_disk_storage_engine=MYISAM`，则尝试使用磁盘临时表实现 CTE 时会出现错误，因为对于 CTE，用于磁盘内部临时表的存储引擎不能是`MyISAM`。从 MySQL 8.0.16 开始，这不再是一个问题，因为`TempTable`现在总是使用`InnoDB`作为磁盘内部临时表的存储引擎。

如前所述，如果实现了 CTE，则即使多次引用，也只会实现一次。为了表示一次性实现，优化器跟踪输出包含一个`creating_tmp_table`的出现，以及一个或多个`reusing_tmp_table`的出现。

CTE 类似于派生表，其后跟随`materialized_from_subquery`节点的引用。对于多次引用的 CTE 也是如此，因此不会出现`materialized_from_subquery`节点的重复（这会给人一种子查询被多次执行的印象，并产生不必要的冗长输出）。对 CTE 的唯一引用具有完整的`materialized_from_subquery`节点，其中包含其子查询计划的描述。其他引用具有简化的`materialized_from_subquery`节点。相同的想法适用于`TRADITIONAL`格式的`EXPLAIN`输出：其他引用的子查询不会显示。
