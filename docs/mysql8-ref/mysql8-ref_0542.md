> 原文：[`dev.mysql.com/doc/refman/8.0/en/derived-condition-pushdown-optimization.html`](https://dev.mysql.com/doc/refman/8.0/en/derived-condition-pushdown-optimization.html)

#### 10.2.2.5 派生条件下推优化

MySQL 8.0.22 及更高版本支持对符合条件的子查询进行派生条件下推。对于诸如`SELECT * FROM (SELECT i, j FROM t1) AS dt WHERE i > *`constant`*`这样的查询，在许多情况下可以将外部`WHERE`条件下推到派生表，从而得到`SELECT * FROM (SELECT i, j FROM t1 WHERE i > *`constant`*) AS dt`。当无法将派生表合并到外部查询中（例如，如果派生表使用聚合），将外部`WHERE`条件下推到派生表应该减少需要处理的行数，从而加快查询的执行速度。

注意

在 MySQL 8.0.22 之前，如果派生表被实体化但未合并，MySQL 会实体化整个表，然后使用`WHERE`条件限定所有结果行。如果未启用派生条件下推或由于其他原因无法使用，则仍然如此。

外部`WHERE`条件可以在以下情况下下推到派生实体化表：

+   当派生表不使用聚合或窗口函数时，外部`WHERE`条件可以直接下推到它。这包括具有多个谓词用`AND`、`OR`或两者连接的`WHERE`条件。

    例如，查询`SELECT * FROM (SELECT f1, f2 FROM t1) AS dt WHERE f1 < 3 AND f2 > 11`被重写为`SELECT f1, f2 FROM (SELECT f1, f2 FROM t1 WHERE f1 < 3 AND f2 > 11) AS dt`。

+   当派生表具有`GROUP BY`且不使用窗口函数时，引用不属于`GROUP BY`的一个或多个列的外部`WHERE`条件可以作为`HAVING`条件下推到派生表。

    例如，`SELECT * FROM (SELECT i, j, SUM(k) AS sum FROM t1 GROUP BY i, j) AS dt WHERE sum > 100`在派生条件下推后被重写为`SELECT * FROM (SELECT i, j, SUM(k) AS sum FROM t1 GROUP BY i, j HAVING sum > 100) AS dt`。

+   当派生表使用`GROUP BY`且外部`WHERE`条件中的列是`GROUP BY`列时，引用这些列的`WHERE`条件可以直接下推到派生表。

    例如，查询`SELECT * FROM (SELECT i,j, SUM(k) AS sum FROM t1 GROUP BY i,j) AS dt WHERE i > 10`被重写为`SELECT * FROM (SELECT i,j, SUM(k) AS sum FROM t1 WHERE i > 10 GROUP BY i,j) AS dt`。

    如果外部`WHERE`条件中有引用`GROUP BY`列的谓词以及引用不是`GROUP BY`列的谓词，前一种类型的谓词被推送为`WHERE`条件，而后一种类型的谓词被推送为`HAVING`条件。例如，在查询`SELECT * FROM (SELECT i, j, SUM(k) AS sum FROM t1 GROUP BY i,j) AS dt WHERE i > 10 AND sum > 100`中，外部`WHERE`子句中的谓词`i > 10`引用了一个`GROUP BY`列，而谓词`sum > 100`则不引用任何`GROUP BY`列。因此，派生表推送优化导致查询被重写为类似于下面所示的方式：

    ```sql
    SELECT * FROM (
        SELECT i, j, SUM(k) AS sum FROM t1
            WHERE i > 10
            GROUP BY i, j
            HAVING sum > 100
        ) AS dt;
    ```

要启用派生条件推送，必须将`optimizer_switch`系统变量的`derived_condition_pushdown`标志（在此版本中添加）设置为`on`，这是默认设置。如果`optimizer_switch`禁用了这个优化，可以使用`DERIVED_CONDITION_PUSHDOWN`优化提示为特定查询启用它。要禁用给定查询的优化，使用`NO_DERIVED_CONDITION_PUSHDOWN`优化提示。

派生表条件推送优化受到以下限制和限制：

+   优化不能在派生表包含`UNION`的情况下使用。这个限制在 MySQL 8.0.29 中被取消。考虑两个表`t1`和`t2`，以及一个包含它们联合的视图`v`，如下所示创建：

    ```sql
    CREATE TABLE t1 (
      id INT NOT NULL AUTO_INCREMENT PRIMARY KEY, 
      c1 INT, 
      KEY i1 (c1)
    );

    CREATE TABLE t2 (
      id INT NOT NULL AUTO_INCREMENT PRIMARY KEY, 
      c1 INT, 
      KEY i1 (c1)
    );

    CREATE OR REPLACE VIEW v AS
         SELECT id, c1 FROM t1
         UNION ALL
         SELECT id, c1 FROM t2;
    ```

    如`EXPLAIN`的输出所示，查询顶层中存在的条件，例如`SELECT * FROM v WHERE c1 = 12`现在可以被推送到派生表中的两个查询块：

    ```sql
    mysql> EXPLAIN FORMAT=TREE SELECT * FROM v WHERE c1 = 12\G
    *************************** 1\. row ***************************
    EXPLAIN: -> Table scan on v  (cost=1.26..2.52 rows=2)
     -> Union materialize  (cost=2.16..3.42 rows=2)
      *-> Covering index lookup on t1 using i1 (c1=12)  (cost=0.35 rows=1)* *-> Covering index lookup on t2 using i1 (c1=12)  (cost=0.35 rows=1)* 1 row in set (0.00 sec)
    ```

    在 MySQL 8.0.29 及更高版本中，可以在`UNION`查询中使用派生表条件推送优化，但有以下例外：

    +   如果`UNION`查询中的任何实现的派生表是递归公共表达式（参见递归公共表达式），则不能使用条件推送。

    +   包含非确定性表达式的条件不能被推送到派生表。

+   派生表不能使用`LIMIT`子句。

+   包含子查询的条件不能被推送。

+   如果派生表是外连接的内部表，则不能使用优化。

+   如果一个实现的派生表是一个公共表达式，如果它被多次引用，则条件不会被推送到它。

+   如果条件使用参数，并且条件形式为`*`derived_column`* > ?`，则条件可以被下推。如果外部`WHERE`条件中的派生列是在基础派生表中具有`?`的表达式，则此条件无法被下推。

+   对于在使用`ALGORITHM=TEMPTABLE`创建的视图的表上而不是在视图本身上的条件查询，多重相等性在解析时不被识别，因此条件无法被下推。这是因为，在优化查询时，条件下推发生在解析阶段，而多重相等性传播发生在优化阶段。

    对于使用`ALGORITHM=MERGE`的视图，这种情况并不是问题，其中相等性可以被传播，条件可以被下推。

+   从 MySQL 8.0.28 开始，如果派生表的`SELECT`列表包含对用户变量的任何赋值，则条件无法被下推。（Bug #104918）
