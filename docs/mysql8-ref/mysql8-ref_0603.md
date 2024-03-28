# 10.9.2 可切换的优化

> 原文：[`dev.mysql.com/doc/refman/8.0/en/switchable-optimizations.html`](https://dev.mysql.com/doc/refman/8.0/en/switchable-optimizations.html)

`optimizer_switch`系统变量可控制优化器行为。其值是一组标志，每个标志的值为`on`或`off`，表示相应的优化器行为是否启用或禁用。此变量具有全局和会话值，并且可以在运行时更改。全局默认值可以在服务器启动时设置。

要查看当前设置的优化器标志集，请选择变量值：

```sql
mysql> SELECT @@optimizer_switch\G
*************************** 1\. row ***************************
@@optimizer_switch: index_merge=on,index_merge_union=on,
                    index_merge_sort_union=on,index_merge_intersection=on,
                    engine_condition_pushdown=on,index_condition_pushdown=on,
                    mrr=on,mrr_cost_based=on,block_nested_loop=on,
                    batched_key_access=off,materialization=on,semijoin=on,
                    loosescan=on,firstmatch=on,duplicateweedout=on,
                    subquery_materialization_cost_based=on,
                    use_index_extensions=on,condition_fanout_filter=on,
                    derived_merge=on,use_invisible_indexes=off,skip_scan=on,
                    hash_join=on,subquery_to_derived=off,
                    prefer_ordering_index=on,hypergraph_optimizer=off,
                    derived_condition_pushdown=on 1 row in set (0.00 sec)
```

要更改`optimizer_switch`的值，请分配一个由一个或多个命令的逗号分隔列表组成的值：

```sql
SET [GLOBAL|SESSION] optimizer_switch='*command*[,*command*]...';
```

每个*`command`*值应具有以下表中所示的形式之一。

| 命令语法 | 含义 |
| --- | --- |
| `default` | 将每个优化重置为其默认值 |
| `*`opt_name`*=default` | 将指定的优化设置为其默认值 |
| `*`opt_name`*=off` | 禁用指定的优化 |
| `*`opt_name`*=on` | 启用指定的优化 |

命令值的顺序无关紧要，尽管如果存在`default`命令，则首先执行该命令。将*`opt_name`*标志设置为`default`会将其设置为`on`或`off`中的默认值。在值中多次指定任何给定的*`opt_name`*是不允许的，并会导致错误。值中的任何错误都会导致分配失败并显示错误，使`optimizer_switch`的值保持不变。

以下列表描述了按优化策略分组的允许的*`opt_name`*标志名称：

+   批量键访问标志

    +   `batched_key_access`（默认为`off`）

        控制使用 BKA 连接算法。

    当设置为`on`时，`batched_key_access`要产生任何效果，`mrr`标志也必须为`on`。目前，对于 MRR 的成本估算过于悲观。因此，还需要将`mrr_cost_based`设置为`off`才能使用 BKA。

    有关更多信息，请参见第 10.2.1.12 节，“块嵌套循环和批量键访问连接”。

+   块嵌套循环标志

    +   `block_nested_loop`（默认为`on`）

        控制使用 BNL 连接算法。在 MySQL 8.0.18 及更高版本中，这也控制使用哈希连接，就像`BNL`和`NO_BNL`优化提示一样。在 MySQL 8.0.20 及更高版本中，MySQL 服务器中移除了块嵌套循环支持，此标志仅控制使用哈希连接，就像引用的优化提示一样。

    欲了解更多信息，请参阅 Section 10.2.1.12，“块嵌套循环和批量键访问连接”。

+   条件过滤标志

    +   `condition_fanout_filter`（默认`on`）

        控制条件过滤的使用。

    欲了解更多信息，请参阅 Section 10.2.1.13，“条件过滤”。

+   派生条件下推标志

    +   `derived_condition_pushdown`（默认`on`）

        控制派生条件下推。

    欲了解更多信息，请参阅 Section 10.2.2.5，“派生条件下推优化”

+   派生表合并标志

    +   `derived_merge`（默认`on`）

        控制将派生表和视图合并到外部查询块中。

    `derived_merge`标志控制优化器是否尝试将派生表、视图引用和公共表达式合并到外部查询块中，假设没有其他规则阻止合并；例如，视图的`ALGORITHM`指令优先于`derived_merge`设置。默认情况下，该标志为`on`以启用合并。

    欲了解更多信息，请参阅 Section 10.2.2.4，“优化派生表、视图引用和公共表达式的合并或材料化”。

+   引擎条件下推标志

    +   `engine_condition_pushdown`（默认`on`）

        控制引擎条件下推。

    欲了解更多信息，请参阅 Section 10.2.1.5，“引擎条件下推优化”。

+   哈希连接标志

    +   `hash_join`（默认`on`）

        仅在 MySQL 8.0.18 中控制哈希连接，对任何后续版本均无效。在 MySQL 8.0.19 及更高版本中，要控制哈希连接的使用，请使用`block_nested_loop`标志。

    更多信息，请参见第 10.2.1.4 节，“哈希连接优化”。

+   索引条件下推标志

    +   `index_condition_pushdown`（默认为`on`）

        控制索引条件下推。

    更多信息，请参见第 10.2.1.6 节，“索引条件下推优化”��

+   索引扩展标志

    +   `use_index_extensions`（默认为`on`）

        控制使用索引扩展。

    更多信息，请参见第 10.3.10 节，“使用索引扩展”。

+   索引合并标志

    +   `index_merge`（默认为`on`）

        控制所有索引合并优化。

    +   `index_merge_intersection`（默认为`on`）

        控制索引合并交集访问优化。

    +   `index_merge_sort_union`（默认为`on`）

        控制索引合并排序-联合访问优化。

    +   `index_merge_union`（默认为`on`）

        控制索引合并联合访问优化。

    更多信息，请参见第 10.2.1.3 节，“索引合并优化”。

+   索引可见性标志

    +   `use_invisible_indexes`（默认为`off`）

        控制使用不可见索引。

    更多信息，请参见第 10.3.12 节，“不可见索引”。

+   限制优化标志

    +   `prefer_ordering_index`（默认为`on`）

        控制是否在查询具有带有`ORDER BY`或`GROUP BY`和`LIMIT`子句的情况下，优化器尝试使用有序索引而不是无序索引、文件排序或其他一些优化。只要优化器确定使用它可以加快查询的执行速度，此优化就会默认执行。

        由于进行此决定的算法无法处理每种可能的情况（部分原因是假设数据分布总是或多或少均匀的），因此存在这种优化可能不可取的情况。在 MySQL 8.0.21 之前，不可能禁用此优化，但在 MySQL 8.0.21 及更高版本中，虽然它仍然是默认行为，但可以通过将`prefer_ordering_index`标志设置为`off`来禁用它。

    有关更多信息和示例，请参见第 10.2.1.19 节，“LIMIT 查询优化”。

+   多范围读取标志

    +   `mrr`（默认`on`）

        控制多范围读取策略。

    +   `mrr_cost_based`（默认`on`）

        如果`mrr=on`，则控制基于成本的 MRR 的使用。

    更多信息，请参见第 10.2.1.11 节，“多范围读取优化”。

+   半连接标志

    +   `duplicateweedout`（默认`on`）

        控制半连接 Duplicate Weedout 策略。

    +   `firstmatch`（默认`on`）

        控制半连接 FirstMatch 策略。

    +   `loosescan`（默认`on`）

        控制半连接松散扫描策略（不要与`GROUP BY`的松散索引扫描混淆）。

    +   `semijoin`（默认`on`）

        控制所有半连接策略。

        在 MySQL 8.0.17 及更高版本中，这也适用于反连接优化。

    `semijoin`、`firstmatch`、`loosescan`和`duplicateweedout`标志可控制半连接策略。`semijoin`标志控制是否使用半连接。如果设置为`on`，则`firstmatch`和`loosescan`标志可更精细地控制允许的半连接策略。

    如果禁用了`duplicateweedout`半连接策略，则除非所有其他适用策略也被禁用，否则不会使用。

    如果`semijoin`和`materialization`都为`on`，则半连接也在适用的情况下使用材料化。这些标志默认为`on`。

    更多信息，请参见第 10.2.2.1 节，“使用半连接转换优化 IN 和 EXISTS 子查询谓词”。

+   跳过扫描标志

    +   `skip_scan`（默认`on`）

        控制跳过扫描访问方法的使用。

    更多信息，请参见跳过扫描范围访问方法。

+   子查询材料化标志

    +   `materialization`（默认`on`）

        控制材料化（包括半连接材料化）。

    +   `subquery_materialization_cost_based`（默认`on`）

        使用基于成本的物化选择。

    `materialization`标志控制是否使用子查询物化。如果`semijoin`和`materialization`都为`on`，半连接也在适用的情况下使用物化。这些标志默认为`on`。

    `subquery_materialization_cost_based`标志允许控制子查询物化和`IN`到`EXISTS`子查询转换之间的选择。如果标志为`on`（默认值），优化器在子查询物化和`IN`到`EXISTS`子查询转换之间执行基于成本的选择，如果可以使用任一方法。如果标志为`off`，优化器选择子查询物化而不是`IN`到`EXISTS`子查询转换。

    更多信息，请参见 Section 10.2.2, “Optimizing Subqueries, Derived Tables, View References, and Common Table Expressions”。

+   子查询转换标志

    +   `subquery_to_derived`（默认为`off`）

        从 MySQL 8.0.21 开始，优化器在许多情况下能够将`SELECT`、`WHERE`、`JOIN`或`HAVING`子句中的标量子查询转换为派生表上的左外连接。（根据派生表的可空性，有时可以进一步简化为内连接。）这可以用于满足以下条件的子查询：

        +   子查询不使用任何非确定性函数，如`RAND()`。

        +   子查询不是可以重写为`MIN()`或`MAX()`的`ANY`或`ALL`子查询。

        +   父查询不设置用户变量，因为重写它可能会影响执行顺序，如果变量在同一查询中被访问多次，可能会导致意外结果。

        +   子查询不应该是相关的，也就是说，它不应该引用外部查询中的列，或包含在外部查询中评估的聚合。

        在 MySQL 8.0.22 之前，子查询不能包含`GROUP BY`子句。

        这种优化也可以应用于作为`IN`、`NOT IN`、`EXISTS`或`NOT EXISTS`参数的表子查询，不包含`GROUP BY`。

        该标志的默认值为`off`，因为在大多数情况下，启用此优化并不会产生明显的性能改进（在许多情况下甚至可能使查询运行更慢），但您可以通过将`subquery_to_derived`标志设置为`on`来启用该优化。它主要用于测试。

        示例，使用标量子查询：

        ```sql
        d
        mysql> CREATE TABLE t1(a INT);

        mysql> CREATE TABLE t2(a INT);

        mysql> INSERT INTO t1 VALUES ROW(1), ROW(2), ROW(3), ROW(4);

        mysql> INSERT INTO t2 VALUES ROW(1), ROW(2);

        mysql> SELECT * FROM t1
         ->     WHERE t1.a > (SELECT COUNT(a) FROM t2);
        +------+
        | a    |
        +------+
        |    3 |
        |    4 |
        +------+

        mysql> SELECT @@optimizer_switch LIKE '%subquery_to_derived=off%';
        +-----------------------------------------------------+
        | @@optimizer_switch LIKE '%subquery_to_derived=off%' |
        +-----------------------------------------------------+
        |                                                   1 |
        +-----------------------------------------------------+

        mysql> EXPLAIN SELECT * FROM t1 WHERE t1.a > (SELECT COUNT(a) FROM t2)\G
        *************************** 1\. row ***************************
                   id: 1
          select_type: PRIMARY
                table: t1
           partitions: NULL
                 type: ALL
        possible_keys: NULL
                  key: NULL
              key_len: NULL
                  ref: NULL
                 rows: 4
             filtered: 33.33
                Extra: Using where
        *************************** 2\. row ***************************
                   id: 2
          select_type: SUBQUERY
                table: t2
           partitions: NULL
                 type: ALL
        possible_keys: NULL
                  key: NULL
              key_len: NULL
                  ref: NULL
                 rows: 2
             filtered: 100.00
                Extra: NULL 
        mysql> SET @@optimizer_switch='subquery_to_derived=on';

        mysql> SELECT @@optimizer_switch LIKE '%subquery_to_derived=off%';
        +-----------------------------------------------------+
        | @@optimizer_switch LIKE '%subquery_to_derived=off%' |
        +-----------------------------------------------------+
        |                                                   0 |
        +-----------------------------------------------------+

        mysql> SELECT @@optimizer_switch LIKE '%subquery_to_derived=on%';
        +----------------------------------------------------+
        | @@optimizer_switch LIKE '%subquery_to_derived=on%' |
        +----------------------------------------------------+
        |                                                  1 |
        +----------------------------------------------------+

        mysql> EXPLAIN SELECT * FROM t1 WHERE t1.a > (SELECT COUNT(a) FROM t2)\G
        *************************** 1\. row ***************************
                   id: 1
          select_type: PRIMARY
                table: <derived2>
           partitions: NULL
                 type: ALL
        possible_keys: NULL
                  key: NULL
              key_len: NULL
                  ref: NULL
                 rows: 1
             filtered: 100.00
                Extra: NULL
        *************************** 2\. row ***************************
                   id: 1
          select_type: PRIMARY
                table: t1
           partitions: NULL
                 type: ALL
        possible_keys: NULL
                  key: NULL
              key_len: NULL
                  ref: NULL
                 rows: 4
             filtered: 33.33
                Extra: Using where; Using join buffer (hash join)
        *************************** 3\. row ***************************
                   id: 2
          select_type: DERIVED
                table: t2
           partitions: NULL
                 type: ALL
        possible_keys: NULL
                  key: NULL
              key_len: NULL
                  ref: NULL
                 rows: 2
             filtered: 100.00
                Extra: NULL
        ```

        从第二个`EXPLAIN`语句后立即执行`SHOW WARNINGS`可以看出，在启用优化的情况下，查询`SELECT * FROM t1 WHERE t1.a > (SELECT COUNT(a) FROM t2)`被重写成类似于这里显示的形式：

        ```sql
        SELECT t1.a FROM t1
            JOIN  ( SELECT COUNT(t2.a) AS c FROM t2 ) AS d
                    WHERE t1.a > d.c;
        ```

        示例，使用带有`IN (*`子查询`*)`的查询：

        ```sql
        mysql> DROP TABLE IF EXISTS t1, t2;

        mysql> CREATE TABLE t1 (a INT, b INT);
        mysql> CREATE TABLE t2 (a INT, b INT);

        mysql> INSERT INTO t1 VALUES ROW(1,10), ROW(2,20), ROW(3,30);
        mysql> INSERT INTO t2
         ->    VALUES ROW(1,10), ROW(2,20), ROW(3,30), ROW(1,110), ROW(2,120), ROW(3,130);

        mysql> SELECT * FROM t1
         ->     WHERE   t1.b < 0
         ->             OR
         ->             t1.a IN (SELECT t2.a + 1 FROM t2);
        +------+------+
        | a    | b    |
        +------+------+
        |    2 |   20 |
        |    3 |   30 |
        +------+------+

        mysql> SET @@optimizer_switch="subquery_to_derived=off";

        mysql> EXPLAIN SELECT * FROM t1
         ->             WHERE   t1.b < 0
         ->                     OR
         ->                     t1.a IN (SELECT t2.a + 1 FROM t2)\G
        *************************** 1\. row ***************************
                   id: 1
          select_type: PRIMARY
                table: t1
           partitions: NULL
                 type: ALL
        possible_keys: NULL
                  key: NULL
              key_len: NULL
                  ref: NULL
                 rows: 3
             filtered: 100.00
                Extra: Using where
        *************************** 2\. row ***************************
                   id: 2
          select_type: DEPENDENT SUBQUERY
                table: t2
           partitions: NULL
                 type: ALL
        possible_keys: NULL
                  key: NULL
              key_len: NULL
                  ref: NULL
                 rows: 6
             filtered: 100.00
                Extra: Using where 
        mysql> SET @@optimizer_switch="subquery_to_derived=on";

        mysql> EXPLAIN SELECT * FROM t1
         ->             WHERE   t1.b < 0
         ->                     OR
         ->                     t1.a IN (SELECT t2.a + 1 FROM t2)\G
        *************************** 1\. row ***************************
                   id: 1
          select_type: PRIMARY
                table: t1
           partitions: NULL
                 type: ALL
        possible_keys: NULL
                  key: NULL
              key_len: NULL
                  ref: NULL
                 rows: 3
             filtered: 100.00
                Extra: NULL
        *************************** 2\. row ***************************
                   id: 1
          select_type: PRIMARY
                table: <derived2>
           partitions: NULL
                 type: ref
        possible_keys: <auto_key0>
                  key: <auto_key0>
              key_len: 9
                  ref: std2.t1.a
                 rows: 2
             filtered: 100.00
                Extra: Using where; Using index
        *************************** 3\. row ***************************
                   id: 2
          select_type: DERIVED
                table: t2
           partitions: NULL
                 type: ALL
        possible_keys: NULL
                  key: NULL
              key_len: NULL
                  ref: NULL
                 rows: 6
             filtered: 100.00
                Extra: Using temporary
        ```

        在这个查询上执行`EXPLAIN`后，检查并简化`SHOW WARNINGS`的结果显示，当启用`subquery_to_derived`标志时，`SELECT * FROM t1 WHERE t1.b < 0 OR t1.a IN (SELECT t2.a + 1 FROM t2)`被重写成类似于这里显示的形式：

        ```sql
        SELECT a, b FROM t1
            LEFT JOIN (SELECT DISTINCT a + 1 AS e FROM t2) d
            ON t1.a = d.e
            WHERE   t1.b < 0
                    OR
                    d.e IS NOT NULL;
        ```

        示例，使用带有`EXISTS (*`子查询`*)`的查询，并与前一个示例中相同的表和数据：

        ```sql
        mysql> SELECT * FROM t1
         ->     WHERE   t1.b < 0
         ->             OR
         ->             EXISTS(SELECT * FROM t2 WHERE t2.a = t1.a + 1);
        +------+------+
        | a    | b    |
        +------+------+
        |    1 |   10 |
        |    2 |   20 |
        +------+------+

        mysql> SET @@optimizer_switch="subquery_to_derived=off";

        mysql> EXPLAIN SELECT * FROM t1
         ->             WHERE   t1.b < 0
         ->                     OR
         ->                     EXISTS(SELECT * FROM t2 WHERE t2.a = t1.a + 1)\G
        *************************** 1\. row ***************************
                   id: 1
          select_type: PRIMARY
                table: t1
           partitions: NULL
                 type: ALL
        possible_keys: NULL
                  key: NULL
              key_len: NULL
                  ref: NULL
                 rows: 3
             filtered: 100.00
                Extra: Using where
        *************************** 2\. row ***************************
                   id: 2
          select_type: DEPENDENT SUBQUERY
                table: t2
           partitions: NULL
                 type: ALL
        possible_keys: NULL
                  key: NULL
              key_len: NULL
                  ref: NULL
                 rows: 6
             filtered: 16.67
                Extra: Using where 
        mysql> SET @@optimizer_switch="subquery_to_derived=on";

        mysql> EXPLAIN SELECT * FROM t1
         ->             WHERE   t1.b < 0
         ->                     OR
         ->                     EXISTS(SELECT * FROM t2 WHERE t2.a = t1.a + 1)\G
        *************************** 1\. row ***************************
                   id: 1
          select_type: PRIMARY
                table: t1
           partitions: NULL
                 type: ALL
        possible_keys: NULL
                  key: NULL
              key_len: NULL
                  ref: NULL
                 rows: 3
             filtered: 100.00
                Extra: NULL
        *************************** 2\. row ***************************
                   id: 1
          select_type: PRIMARY
                table: <derived2>
           partitions: NULL
                 type: ALL
        possible_keys: NULL
                  key: NULL
              key_len: NULL
                  ref: NULL
                 rows: 6
             filtered: 100.00
                Extra: Using where; Using join buffer (hash join)
        *************************** 3\. row ***************************
                   id: 2
          select_type: DERIVED
                table: t2
           partitions: NULL
                 type: ALL
        possible_keys: NULL
                  key: NULL
              key_len: NULL
                  ref: NULL
                 rows: 6
             filtered: 100.00
                Extra: Using temporary
        ```

        如果我们在`subquery_to_derived`已启用的情况下，在查询`SELECT * FROM t1 WHERE t1.b < 0 OR EXISTS(SELECT * FROM t2 WHERE t2.a = t1.a + 1)`上运行`EXPLAIN`后执行`SHOW WARNINGS`并简化结果的第二行，我们会看到它被重写成类似于这里显示的形式：

        ```sql
        SELECT a, b FROM t1
        LEFT JOIN (SELECT DISTINCT 1 AS e1, t2.a AS e2 FROM t2) d
        ON t1.a + 1 = d.e2
        WHERE   t1.b < 0
                OR
                d.e1 IS NOT NULL;
        ```

        欲了解更多信息，请参阅 Section 10.2.2.4, “Optimizing Derived Tables, View References, and Common Table Expressions with Merging or Materialization”，以及 Section 10.2.1.19, “LIMIT Query Optimization”，和 Section 10.2.2.1, “Optimizing IN and EXISTS Subquery Predicates with Semijoin Transformations”。

当您为`optimizer_switch`分配一个值时，未提及的标志保持其当前值。这使得可以在单个语句中启用或禁用特定的优化器行为，而不影响其他行为。该语句不依赖于其他优化器标志的存在及其值是什么。假设所有索引合并优化都已启用：

```sql
mysql> SELECT @@optimizer_switch\G
*************************** 1\. row ***************************
@@optimizer_switch: index_merge=on,index_merge_union=on,
                    index_merge_sort_union=on,index_merge_intersection=on,
                    engine_condition_pushdown=on,index_condition_pushdown=on,
                    mrr=on,mrr_cost_based=on,block_nested_loop=on,
                    batched_key_access=off,materialization=on,semijoin=on,
                    loosescan=on, firstmatch=on,
                    subquery_materialization_cost_based=on,
                    use_index_extensions=on,condition_fanout_filter=on,
                    derived_merge=on,use_invisible_indexes=off,skip_scan=on,
                    hash_join=on,subquery_to_derived=off,
                    prefer_ordering_index=on
```

如果服务器对某些查询使用了索引合并联合或索引合并排序-联合访问方法，并且您想检查优化器在没有它们的情况下是否可以表现更好，请像这样设置变量值：

```sql
mysql> SET optimizer_switch='index_merge_union=off,index_merge_sort_union=off';

mysql> SELECT @@optimizer_switch\G
*************************** 1\. row ***************************
@@optimizer_switch: index_merge=on,index_merge_union=off,
                    index_merge_sort_union=off,index_merge_intersection=on,
                    engine_condition_pushdown=on,index_condition_pushdown=on,
                    mrr=on,mrr_cost_based=on,block_nested_loop=on,
                    batched_key_access=off,materialization=on,semijoin=on,
                    loosescan=on, firstmatch=on,
                    subquery_materialization_cost_based=on,
                    use_index_extensions=on,condition_fanout_filter=on,
                    derived_merge=on,use_invisible_indexes=off,skip_scan=on,
                    hash_join=on,subquery_to_derived=off,
                    prefer_ordering_index=on
```
