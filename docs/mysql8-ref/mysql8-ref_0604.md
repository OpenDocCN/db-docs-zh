# 10.9.3 优化提示

> 原文：[`dev.mysql.com/doc/refman/8.0/en/optimizer-hints.html`](https://dev.mysql.com/doc/refman/8.0/en/optimizer-hints.html)

控制优化器策略的一种方法是设置`optimizer_switch`系统变量（参见第 10.9.2 节，“可切换的优化”）。对此变量的更改会影响所有后续查询的执行；要使一个查询与另一个查询不同，需要在每个查询之前更改`optimizer_switch`。

控制优化器的另一种方法是使用优化提示，可以在单个语句中指定。因为优化提示是基于每个语句的基础上应用的，所以它们比使用`optimizer_switch`可以更精细地控制语句执行计划。例如，您可以为语句中的一个表启用优化，并为另一个表禁用优化。语句中的提示优先于`optimizer_switch`标志。

示例:

```sql
SELECT /*+ NO_RANGE_OPTIMIZATION(t3 PRIMARY, f2_idx) */ f1
  FROM t3 WHERE f1 > 30 AND f1 < 33;
SELECT /*+ BKA(t1) NO_BKA(t2) */ * FROM t1 INNER JOIN t2 WHERE ...;
SELECT /*+ NO_ICP(t1, t2) */ * FROM t1 INNER JOIN t2 WHERE ...;
SELECT /*+ SEMIJOIN(FIRSTMATCH, LOOSESCAN) */ * FROM t1 ...;
EXPLAIN SELECT /*+ NO_ICP(t1) */ * FROM t1 WHERE ...;
SELECT /*+ MERGE(dt) */ * FROM (SELECT * FROM t1) AS dt;
INSERT /*+ SET_VAR(foreign_key_checks=OFF) */ INTO t2 VALUES(2);
```

此处描述的优化提示与第 10.9.4 节，“索引提示”中描述的索引提示不同。优化提示和索引提示可以单独使用或一起使用。

+   优化提示概述

+   优化提示语法

+   连接顺序优化提示

+   表级别优化提示

+   索引级别优化提示

+   子查询优化提示

+   语句执行时间优化提示

+   变量设置提示语法

+   资源组提示语法

+   用于命名查询块的优化提示

#### 优化提示概述

优化提示适用于不同的作用域级别:

+   全局: 提示影响整个语句

+   查询块: 提示影响语句中的特定查询块

+   表级别: 提示影响查询块内的特定表

+   索引级别: 提示影响表内的特定索引

以下表格总结了可用的优化提示、它们影响的优化策略以及适用的范围。更多细节稍后给出。

**表 10.2 可用的优化提示**

| 提示名称 | 描述 | 适用范围 |
| --- | --- | --- |
| `BKA`, `NO_BKA` | 影响批量键访问连接处理 | 查询块，表 |
| `BNL`, `NO_BNL` | 在 MySQL 8.0.20 之前：影响块嵌套循环连接处理；MySQL 8.0.18 及更高版本：还影响哈希连接优化；MySQL 8.0.20 及更高版本：仅影响哈希连接优化 | 查询块，表 |
| `DERIVED_CONDITION_PUSHDOWN`, `NO_DERIVED_CONDITION_PUSHDOWN` | 使用或忽略对物化派生表的派生条件推送优化（MySQL 8.0.22 中添加） | 查询块，表 |
| `GROUP_INDEX`, `NO_GROUP_INDEX` | 使用或忽略指定的索引或索引进行`GROUP BY`操作中的索引扫描（MySQL 8.0.20 中添加） | 索引 |
| `HASH_JOIN`, `NO_HASH_JOIN` | 影响哈希连接优化（仅适用于 MySQL 8.0.18） | 查询块，表 |
| `INDEX`, `NO_INDEX` | 作为`JOIN_INDEX`, `GROUP_INDEX`, 和 `ORDER_INDEX` 的组合，或者`NO_JOIN_INDEX`, `NO_GROUP_INDEX`, 和 `NO_ORDER_INDEX` 的组合（MySQL 8.0.20 中添加） | 索引 |
| `INDEX_MERGE`, `NO_INDEX_MERGE` | 影响索引合并优化 | 表，索引 |
| `JOIN_FIXED_ORDER` | 使用 `FROM` 子句中指定的表顺序作为连接顺序 | 查询块 |
| `JOIN_INDEX`, `NO_JOIN_INDEX` | 使用或忽略指定的索引或索引以供任何访问方法使用（MySQL 8.0.20 中添加） | 索引 |
| `JOIN_ORDER` | 使用提示中指定的表顺序作为连接顺序 | 查询块 |
| `JOIN_PREFIX` | 使用提示中指定的表顺序作为连接顺序的第一个表 | 查询块 |
| `JOIN_SUFFIX` | 使用提示中指定的表顺序作为连接顺序的最后表 | 查询块 |
| `MAX_EXECUTION_TIME` | 限制语句执行时间 | 全局 |
| `MERGE`, `NO_MERGE` | 影响派���表/视图合并到外部查询块 | 表 |
| `MRR`, `NO_MRR` | 影响多范围读取优化 | 表，索引 |
| `NO_ICP` | 影响索引条件下推优化 | 表，索引 |
| `NO_RANGE_OPTIMIZATION` | 影响范围优化 | 表，索引 |
| `ORDER_INDEX`, `NO_ORDER_INDEX` | 使用或忽略指定的索引或索引以供对行进行排序（MySQL 8.0.20 中添加） | 索引 |
| `QB_NAME` | 为查询块分配名称 | 查询块 |
| `RESOURCE_GROUP` | 在语句执行期间设置资源组 | 全局 |
| `SEMIJOIN`, `NO_SEMIJOIN` | 影响半连接策略；从 MySQL 8.0.17 开始，这也适用于反连接 | 查询块 |
| `SKIP_SCAN`, `NO_SKIP_SCAN` | 影响跳过扫描优化 | 表，索引 |
| `SET_VAR` | 在语句执行期间设置变量 | 全局 |
| `SUBQUERY` | 影响物化，`IN`-to-`EXISTS`子查询策略 | 查询块 |
| 提示名称 | 描述 | 适用范围 |

禁用优化会阻止优化器使用它。启用优化意味着优化器可以自由使用该策略，如果它适用于语句执行，而不是优化器一定会使用它。

#### 优化器提示语法

MySQL 支持 SQL 语句中的注释，如第 11.7 节，“注释”所述。优化器提示必须在`/*+ ... */`注释中指定。也就是说，优化器提示使用一种`/* ... */` C 风格注释语法的变体，其中在`/*`注释开头序列后跟着一个`+`字符。例如：

```sql
/*+ BKA(t1) */
/*+ BNL(t1, t2) */
/*+ NO_RANGE_OPTIMIZATION(t4 PRIMARY) */
/*+ QB_NAME(qb2) */
```

在`+`字符后允许空格。

解析器在`SELECT`，`UPDATE`，`INSERT`，`REPLACE`和`DELETE`语句的初始关键字后识别优化器提示注释。在这些上下文中允许使用提示：

+   在查询和数据更改语句的开头：

    ```sql
    SELECT /*+ ... */ ...
    INSERT /*+ ... */ ...
    REPLACE /*+ ... */ ...
    UPDATE /*+ ... */ ...
    DELETE /*+ ... */ ...
    ```

+   在查询块的开头：

    ```sql
    (SELECT /*+ ... */ ... )
    (SELECT ... ) UNION (SELECT /*+ ... */ ... )
    (SELECT /*+ ... */ ... ) UNION (SELECT /*+ ... */ ... )
    UPDATE ... WHERE x IN (SELECT /*+ ... */ ...)
    INSERT ... SELECT /*+ ... */ ...
    ```

+   在以`EXPLAIN`为前缀的可提示语句中。例如：

    ```sql
    EXPLAIN SELECT /*+ ... */ ...
    EXPLAIN UPDATE ... WHERE x IN (SELECT /*+ ... */ ...)
    ```

    这意味着您可以使用`EXPLAIN`查看优化器提示如何影响执行计划。在`EXPLAIN`之后立即使用`SHOW WARNINGS`查看提示的使用情况。由后续的`SHOW WARNINGS`显示的扩展`EXPLAIN`输出指示使用了哪些提示。被忽略的提示不会显示。

一个提示注释可以包含多个提示，但一个查询块不能包含多个提示注释。这是有效的：

```sql
SELECT /*+ BNL(t1) BKA(t2) */ ...
```

但这是无效的：

```sql
SELECT /*+ BNL(t1) */ /* BKA(t2) */ ...
```

当一个提示注释包含多个提示时，存在重复和冲突的可能性。以下是一般准则。对于特定的提示类型，可能会适用额外的规则，如提示描述中所示。

+   重复的提示：对于像`/*+ MRR(idx1) MRR(idx1) */`这样的提示，MySQL 使用第一个提示，并发出关于重复提示的警告。

+   冲突的提示：对于像`/*+ MRR(idx1) NO_MRR(idx1) */`这样的提示，MySQL 使用第一个提示，并发出关于第二个冲突提示的警告。

查询块名称是标识符，遵循关于哪些名称有效以及如何引用它们的通常规则（参见第 11.2 节，“模式对象名称”）。

提示名称、查询块名称和策略名称不区分大小写。表和索引名称的引用遵循通常的标识符大小写敏感性规则（参见第 11.2.3 节，“标识符大小写敏感性”）。

#### 连接顺序优化提示

连接顺序提示影响优化器连接表的顺序。

`JOIN_FIXED_ORDER`提示的语法：

```sql
*hint_name*([@*query_block_name*])
```

其他连接顺序提示的语法：

```sql
*hint_name*([@*query_block_name*] *tbl_name* [, *tbl_name*] ...)
*hint_name*(*tbl_name*[@*query_block_name*] [, *tbl_name*[@*query_block_name*]] ...)
```

语法指的是这些术语：

+   *`hint_name`*：允许使用这些提示名称：

    +   `JOIN_FIXED_ORDER`：强制优化器按照它们在`FROM`子句中出现的顺序连接表。这与指定`SELECT STRAIGHT_JOIN`相同。

    +   `JOIN_ORDER`：指示优化器使用指定的表顺序连接表。提示适用于指定的表。优化器可以在连接顺序中放置未命名的表，包括在指定表之间。

    +   `JOIN_PREFIX`：指示优化器使用连接执行计划的第一个表的指定表顺序连接表。提示适用于指定的表。优化器将所有其他表放在指定表之后。

    +   `JOIN_SUFFIX`：指示优化器使用指定的表顺序连接连接执行计划的最后表。提示适用于指定的表。优化器将所有其他表放在指定表之前。

+   *`tbl_name`*：语句中使用的表的名称。指定表名称的提示适用于所有指定的表。`JOIN_FIXED_ORDER`提示不指定表名，并适用于其出现的查询块的`FROM`子句中的所有表。

    如果表有别名，则提示必须引用别名，而不是表名。

    提示中的表名不能带有模式名限定。

+   *`query_block_name`*：提示适用的查询块。如果提示中不包含前导`@*`query_block_name`*`，则提示适用于其出现的查询块。对于`*`tbl_name`*@*`query_block_name`*`语法，提示适用于指定查询块中的指定表。要为查询块指定名称，请参见为查询块命名的优化提示。

示例：

```sql
SELECT
/*+ JOIN_PREFIX(t2, t5@subq2, t4@subq1)
    JOIN_ORDER(t4@subq1, t3)
    JOIN_SUFFIX(t1) */
COUNT(*) FROM t1 JOIN t2 JOIN t3
           WHERE t1.f1 IN (SELECT /*+ QB_NAME(subq1) */ f1 FROM t4)
             AND t2.f1 IN (SELECT /*+ QB_NAME(subq2) */ f1 FROM t5);
```

提示控制将合并到外部查询块的半连接表的行为。如果子查询 `subq1` 和 `subq2` 被转换为半连接，表 `t4@subq1` 和 `t5@subq2` 将合并到外部查询块。在这种情况下，外部查询块中指定的提示控制 `t4@subq1`、`t5@subq2` 表的行为。

优化器根据以下原则解析连接顺序提示：

+   多个提示实例

    每种类型只应用一个 `JOIN_PREFIX` 和 `JOIN_SUFFIX` 提示。后续相同类型的提示将被忽略，并显示警告。`JOIN_ORDER` 可以多次指定。

    示例：

    ```sql
    /*+ JOIN_PREFIX(t1) JOIN_PREFIX(t2) */
    ```

    第二个 `JOIN_PREFIX` 提示将被忽略，并显示警告。

    ```sql
    /*+ JOIN_PREFIX(t1) JOIN_SUFFIX(t2) */
    ```

    两个提示都适用。不会出现警告。

    ```sql
    /*+ JOIN_ORDER(t1, t2) JOIN_ORDER(t2, t3) */
    ```

    两个提示都适用。不会出现警告。

+   冲突的提示

    在某些情况下，提示可能会发生冲突，例如当 `JOIN_ORDER` 和 `JOIN_PREFIX` 具有不可能同时应用的表顺序时：

    ```sql
    SELECT /*+ JOIN_ORDER(t1, t2) JOIN_PREFIX(t2, t1) */ ... FROM t1, t2;
    ```

    在这种情况下，第一个指定的提示被应用，后续冲突的提示将被忽略，且不会出现警告。一个无法应用的有效提示将被静默忽略，且不会出现警告。

+   被忽略的提示

    如果提示中指定的表存在循环依赖，则提示将被忽略。

    示例：

    ```sql
    /*+ JOIN_ORDER(t1, t2) JOIN_PREFIX(t2, t1) */
    ```

    `JOIN_ORDER` 提示将表 `t2` 设置为依赖于 `t1`。`JOIN_PREFIX` 提示被忽略，因为表 `t1` 不能依赖于 `t2`。被忽略的提示不会显示在扩展的 `EXPLAIN` 输出中。

+   与 `const` 表的交互

    MySQL 优化器将 `const` 表放在连接顺序的首位，`const` 表的位置不受提示影响。在连接顺序提示中忽略对 `const` 表的引用，尽管提示仍然适用。例如，以下两种提示是等效的：

    ```sql
    JOIN_ORDER(t1, *const_tbl*, t2)
    JOIN_ORDER(t1, t2)
    ```

    在扩展的 `EXPLAIN` 输出中显示的接受的提示包括按照指定方式指定的 `const` 表。

+   与连接操作类型的交互

    MySQL 支持几种类型的连接：`LEFT`、`RIGHT`、`INNER`、`CROSS`、`STRAIGHT_JOIN`。与指定的连接类型冲突的提示将被忽略，且不会出现警告。

    示例：

    ```sql
    SELECT /*+ JOIN_PREFIX(t1, t2) */FROM t2 LEFT JOIN t1;
    ```

    在提示中请求的连接顺序与 `LEFT JOIN` 所需的顺序之间发生冲突。提示将被忽略，且不会出现警告。

#### 表级优化提示

表级提示影响：

+   使用块嵌套循环（BNL）和批量键访问（BKA）连接处理算法（参见第 10.2.1.12 节，“块嵌套循环和批量键访问连接”）。

+   衍生表、视图引用或公共表达式是否应合并到外部查询块中，或者使用内部临时表进行材料化。

+   使用衍生表条件下推优化（在 MySQL 8.0.22 中添加）。请参见第 10.2.2.5 节，“衍生条件下推优化”。

这些提示类型适用于特定表或查询块中的所有表。

表级提示的语法：

```sql
*hint_name*([@*query_block_name*] [*tbl_name* [, *tbl_name*] ...])
*hint_name*([*tbl_name*@*query_block_name* [, *tbl_name*@*query_block_name*] ...])
```

语法涉及这些术语：

+   *`hint_name`*：允许使用这些提示名称：

    +   `BKA`, `NO_BKA`：启用或禁用指定表的批量键访问。

    +   `BNL`, `NO_BNL`：启用或禁用指定表的块嵌套循环。在 MySQL 8.0.18 及更高版本中，这些提示还启用和禁用哈希连接优化。

        注意

        MySQL 8.0.20 及更高版本中移除了块嵌套循环优化，但仍支持`BNL`和`NO_BNL`以启用和禁用哈希连接。

    +   `DERIVED_CONDITION_PUSHDOWN`, `NO_DERIVED_CONDITION_PUSHDOWN`：启用或禁用指定表的衍生表条件下推（在 MySQL 8.0.22 中添加）。有关更多信息，请参见第 10.2.2.5 节，“衍生条件下推优化”。

    +   `HASH_JOIN`, `NO_HASH_JOIN`：仅在 MySQL 8.0.18 中启用或禁用指定表的哈希连接。在 MySQL 8.0.19 或更高版本中，这些提示无效，应改用`BNL`或`NO_BNL`。

    +   `MERGE`, `NO_MERGE`：为指定的表、视图引用或公共表达式启用合并；或禁用合并，改为使用材料化。

    注意

    要使用块嵌套循环或批量键访问提示为外连接的任何内部表启用连接缓冲，必须为外连接的所有内部表启用连接缓冲。

+   *`tbl_name`*：语句中使用的表的名称。提示适用于所有命名的表。如果提示未命名任何表，则适用于其出现的查询块中的所有表。

    如果表有别名，则提示必须引用别名，而不是表名。

    提示中的表名不能带有模式名。

+   *`query_block_name`*：提示适用的查询块。如果提示不包含前导`@*`query_block_name`*`，则提示适用于其出现的查询块。对于`*`tbl_name`*@*`query_block_name`*`语法，提示适用于命名查询块中的命名表。要为查询块分配名称，请参见为查询块命名的优化器提示。

示例：

```sql
SELECT /*+ NO_BKA(t1, t2) */ t1.* FROM t1 INNER JOIN t2 INNER JOIN t3;
SELECT /*+ NO_BNL() BKA(t1) */ t1.* FROM t1 INNER JOIN t2 INNER JOIN t3;
SELECT /*+ NO_MERGE(dt) */ * FROM (SELECT * FROM t1) AS dt;
```

表级提示适用于接收来自前面表的记录的表，而不是发送方表。考虑以下语句：

```sql
SELECT /*+ BNL(t2) */ FROM t1, t2;
```

如果优化器选择首先处理`t1`，则通过在开始从`t2`读取之前缓冲来自`t1`的行，将对`t2`应用块嵌套循环连接。如果优化器选择首先处理`t2`，则提示无效，因为`t2`是发送方表。

对于`MERGE`和`NO_MERGE`提示，适用以下优先规则：

+   提示优先于任何不是技术约束的优化器启发式。 (如果提供提示作为建议没有效果，优化器有理由忽略它。)

+   提示优先于`optimizer_switch`系统变量的`derived_merge`标志。

+   对于视图引用，视图定义中的`ALGORITHM={MERGE|TEMPTABLE}`子句优先于在引用视图的查询中指定的提示。

#### 索引级优化器提示

索引级提示影响优化器为特定表或索引使用哪些索引处理策略。这些提示类型影响索引条件下推（ICP）、多范围读取（MRR）、索引合并和范围优化的使用（参见第 10.2.1 节，“优化 SELECT 语句”）。

索引级提示的语法：

```sql
*hint_name*([@*query_block_name*] *tbl_name* [*index_name* [, *index_name*] ...])
*hint_name*(*tbl_name*@*query_block_name* [*index_name* [, *index_name*] ...])
```

语法涉及以下术语：

+   *`hint_name`*：允许使用这些提示名称：

    +   `GROUP_INDEX`, `NO_GROUP_INDEX`：为`GROUP BY`操作启用或禁用指定索引或索引的索引扫描。等同于索引提示`FORCE INDEX FOR GROUP BY`, `IGNORE INDEX FOR GROUP BY`。从 MySQL 8.0.20 及更高版本提供。

    +   `INDEX`, `NO_INDEX`：作为`JOIN_INDEX`, `GROUP_INDEX`, 和 `ORDER_INDEX`的组合，强制服务器使用指定的索引或索引来处理任何范围，或作为`NO_JOIN_INDEX`, `NO_GROUP_INDEX`, 和 `NO_ORDER_INDEX`的组合，导致服务器忽略指定的索引或索引用于任何范围。等同于`FORCE INDEX`, `IGNORE INDEX`。从 MySQL 8.0.20 开始提供。

    +   `INDEX_MERGE`, `NO_INDEX_MERGE`：启用或禁用指定表或索引的索引合并访问方法。有关此访问方法的信息，请参见 Section 10.2.1.3, “Index Merge Optimization”。这些提示适用于所有三种索引合并算法。

        `INDEX_MERGE`提示强制优化器使用指定表的指定索引集进行索引合并。如果未指定索引，则优化器会考虑所有可能的索引组合并选择最经济的一个。如果索引组合不适用于给定的语句，则可能会忽略此提示。

        `NO_INDEX_MERGE`提示禁用涉及任何指定索引的索引合并组合。如果提示未指定任何索引，则不允许对表进行索引合并。

    +   `JOIN_INDEX`, `NO_JOIN_INDEX`: 强制 MySQL 使用或忽略指定的索引或索引，用于任何访问方法，如`ref`、`range`、`index_merge`等。相当于`FORCE INDEX FOR JOIN`、`IGNORE INDEX FOR JOIN`。从 MySQL 8.0.20 版本开始提供。

    +   `MRR`, `NO_MRR`: 启用或禁用指定表或索引的 MRR。MRR 提示仅适用于`InnoDB`和`MyISAM`表。有关此访问方法的信息，请参见 Section 10.2.1.11, “Multi-Range Read Optimization”。

    +   `NO_ICP`: 禁用指定表或索引的 ICP。默认情况下，ICP 是一种候选优化策略，因此没有启用它的提示。有关此访问方法的信息，请参见 Section 10.2.1.6, “Index Condition Pushdown Optimization”。

    +   `NO_RANGE_OPTIMIZATION`: 禁用指定表或索引的索引范围访问。此提示还禁用了表或索引的索引合并和松散索引扫描。默认情况下，范围访问是一种候选优化策略，因此没有启用它的提示。

        当范围数量可能很高且范围优化需要大量资源时，此提示可能会有用。

    +   `ORDER_INDEX`, `NO_ORDER_INDEX`: 导致 MySQL 使用或忽略指定的索引或索引来对行进行排序。相当于`FORCE INDEX FOR ORDER BY`、`IGNORE INDEX FOR ORDER BY`。从 MySQL 8.0.20 版本开始提供。

    +   `SKIP_SCAN`, `NO_SKIP_SCAN`: 启用或禁用指定表或索引的跳过扫描访问方法。有关此访问方法的信息，请参见跳过扫描范围访问方法。这些提示从 MySQL 8.0.13 版本开始提供。

        `SKIP_SCAN`提示强制优化器使用跳过扫描来使用指定的表和指定的索引集。如果未指定索引，则优化器考虑所有可能的索引并选择最经济的一个。如果索引不适用于给定语句，则可能会忽略提示。

        `NO_SKIP_SCAN`提示禁用了指定索引的跳过扫描。如果提示未指定任何索引，则不允许对表进行跳过扫描。

+   *`tbl_name`*：提示适用的表。

+   *`index_name`*：命名表中索引的名称。提示适用于所有命名的索引。如果提示未命名任何索引，则适用于表中的所有索引。

    要引用主键，请使用名称`PRIMARY`。要查看表的索引名称，请使用`SHOW INDEX`。

+   *`query_block_name`*：提示适用的查询块。如果提示中不包含前导`@*`query_block_name`*`，则提示适用于出现在其中的查询块。对于`*`tbl_name`*@*`query_block_name`*`语法，提示适用于命名查询块中的命名表。要为查询块分配名称，请参见为查询块命名的优化器提示。

示例：

```sql
SELECT /*+ INDEX_MERGE(t1 f3, PRIMARY) */ f2 FROM t1
  WHERE f1 = 'o' AND f2 = f3 AND f3 <= 4;
SELECT /*+ MRR(t1) */ * FROM t1 WHERE f2 <= 3 AND 3 <= f3;
SELECT /*+ NO_RANGE_OPTIMIZATION(t3 PRIMARY, f2_idx) */ f1
  FROM t3 WHERE f1 > 30 AND f1 < 33;
INSERT INTO t3(f1, f2, f3)
  (SELECT /*+ NO_ICP(t2) */ t2.f1, t2.f2, t2.f3 FROM t1,t2
   WHERE t1.f1=t2.f1 AND t2.f2 BETWEEN t1.f1
   AND t1.f2 AND t2.f2 + 1 >= t1.f1 + 1);
SELECT /*+ SKIP_SCAN(t1 PRIMARY) */ f1, f2
  FROM t1 WHERE f2 > 40;
```

下面的示例使用了索引合并提示，但其他索引级别的提示遵循相同的原则，关于提示忽略和优化器提示在与`optimizer_switch`系统变量或索引提示的优先级方面的关系。

假设表`t1`具有列`a`、`b`、`c`和`d`；并且在`a`、`b`和`c`上分别存在名为`i_a`、`i_b`和`i_c`的索引：

```sql
SELECT /*+ INDEX_MERGE(t1 i_a, i_b, i_c)*/ * FROM t1
  WHERE a = 1 AND b = 2 AND c = 3 AND d = 4;
```

在这种情况下，索引合并用于`(i_a, i_b, i_c)`。

```sql
SELECT /*+ INDEX_MERGE(t1 i_a, i_b, i_c)*/ * FROM t1
  WHERE b = 1 AND c = 2 AND d = 3;
```

在这种情况下，索引合并用于`(i_b, i_c)`。

```sql
/*+ INDEX_MERGE(t1 i_a, i_b) NO_INDEX_MERGE(t1 i_b) */
```

`NO_INDEX_MERGE`被忽略，因为存在相同表的先前提示。

```sql
/*+ NO_INDEX_MERGE(t1 i_a, i_b) INDEX_MERGE(t1 i_b) */
```

`INDEX_MERGE`被忽略，因为存在相同表的先前提示。

对于`INDEX_MERGE`和`NO_INDEX_MERGE`优化器提示，适用以下优先规则：

+   如果指定了优化器提示并且适用，则它优先于`optimizer_switch`系统变量的与索引合并相关的标志。

    ```sql
    SET optimizer_switch='index_merge_intersection=off';
    SELECT /*+ INDEX_MERGE(t1 i_b, i_c) */ * FROM t1
    WHERE b = 1 AND c = 2 AND d = 3;
    ```

    提示优先于`optimizer_switch`。在这种情况下，索引合并用于`(i_b, i_c)`。

    ```sql
    SET optimizer_switch='index_merge_intersection=on';
    SELECT /*+ INDEX_MERGE(t1 i_b) */ * FROM t1
    WHERE b = 1 AND c = 2 AND d = 3;
    ```

    提示只指定了一个索引，因此不适用，并且 `optimizer_switch` 标志（`on`）适用。如果优化器评估为成本有效，则使用索引合并。

    ```sql
    SET optimizer_switch='index_merge_intersection=off';
    SELECT /*+ INDEX_MERGE(t1 i_b) */ * FROM t1
    WHERE b = 1 AND c = 2 AND d = 3;
    ```

    提示只指定了一个索引，因此不适用，并且 `optimizer_switch` 标志（`off`）适用。不使用索引合并。

+   索引级别的优化提示 `GROUP_INDEX`、`INDEX`、`JOIN_INDEX` 和 `ORDER_INDEX` 都优先于等效的 `FORCE INDEX` 提示；也就是说，它们导致 `FORCE INDEX` 提示被忽略。同样，`NO_GROUP_INDEX`、`NO_INDEX`、`NO_JOIN_INDEX` 和 `NO_ORDER_INDEX` 提示都优先于任何 `IGNORE INDEX` 等效项，也导致它们被忽略。

    索引级别的优化提示 `GROUP_INDEX`、`NO_GROUP_INDEX`、`INDEX`、`NO_INDEX`、`JOIN_INDEX`、`NO_JOIN_INDEX`、`ORDER_INDEX` 和 `NO_ORDER_INDEX` 提示都优先于所有其他优化提示，包括其他索引级别的优化提示。任何其他优化提示仅适用于这些提示允许的索引。

    `GROUP_INDEX`、`INDEX`、`JOIN_INDEX` 和 `ORDER_INDEX` 提示都等同于 `FORCE INDEX`，而不是 `USE INDEX`。这是因为使用这些提示中的一个或多个意味着只有在无法使用命名的索引来查找表中的行时才会使用表扫描。要使 MySQL 使用与给定的 `USE INDEX` 实例相同的索引或索引集，可以使用 `NO_INDEX`、`NO_JOIN_INDEX`、`NO_GROUP_INDEX`、`NO_ORDER_INDEX` 或这些提示的某种组合。

    要复制查询 `SELECT a,c FROM t1 USE INDEX FOR ORDER BY (i_a) ORDER BY a` 中 `USE INDEX` 的效果，可以使用 `NO_ORDER_INDEX` 优化提示来覆盖表上除所需索引之外的所有索引，如下所示：

    ```sql
    SELECT /*+ NO_ORDER_INDEX(t1 i_b,i_c) */ a,c
        FROM t1
        ORDER BY a;
    ```

    尝试将整个表的 `NO_ORDER_INDEX` 与 `USE INDEX FOR ORDER BY` 结合起来不起作用，因为 `NO_ORDER_BY` 导致 `USE INDEX` 被忽略，如下所示：

    ```sql
    mysql> EXPLAIN SELECT /*+ NO_ORDER_INDEX(t1) */ a,c FROM t1
     ->     USE INDEX FOR ORDER BY (i_a) ORDER BY a\G
    *************************** 1\. row ***************************
               id: 1
      select_type: SIMPLE
            table: t1
       partitions: NULL
             type: ALL
    possible_keys: NULL
              key: NULL
          key_len: NULL
              ref: NULL
             rows: 256
         filtered: 100.00
            Extra: Using filesort
    ```

+   `USE INDEX`、`FORCE INDEX` 和 `IGNORE INDEX` 索引提示的优先级高于 `INDEX_MERGE` 和 `NO_INDEX_MERGE` 优化提示。

    ```sql
    /*+ INDEX_MERGE(t1 i_a, i_b, i_c) */ ... IGNORE INDEX i_a
    ```

    `IGNORE INDEX` 优先于 `INDEX_MERGE`，因此索引 `i_a` 被排除在索引合并的可能范围之外。

    ```sql
    /*+ NO_INDEX_MERGE(t1 i_a, i_b) */ ... FORCE INDEX i_a, i_b
    ```

    由于 `FORCE INDEX`，`i_a, i_b` 不允许进行索引合并，但是优化器被强制使用 `i_a` 或 `i_b` 进行 `range` 或 `ref` 访问。没有冲突；两个提示都适用。

+   如果 `IGNORE INDEX` 提示命名多个索引，则这些索引对于索引合并是不可用的。

+   `FORCE INDEX`和`USE INDEX`提示仅使命名的索引可用于索引合并。

    ```sql
    SELECT /*+ INDEX_MERGE(t1 i_a, i_b, i_c) */ a FROM t1
    FORCE INDEX (i_a, i_b) WHERE c = 'h' AND a = 2 AND b = 'b';
    ```

    对于`(i_a, i_b)`，使用索引合并交集访问算法。如果将`FORCE INDEX`更改为`USE INDEX`，情况也是如此。

#### 子查询优化提示

子查询提示影响是否使用半连接转换以及允许哪些半连接策略，并且在不使用半连接时，是否使用子查询材料化或`IN`到`EXISTS`转换。有关这些优化的更多信息，请参见第 10.2.2 节，“优化子查询、派生表、视图引用和公共表表达式”。

影响半连接策略的提示的语法：

```sql
*hint_name*([@*query_block_name*] [*strategy* [, *strategy*] ...])
```

语法涉及以下术语：

+   *`hint_name`*：允许使用这些提示名称：

    +   `SEMIJOIN`，`NO_SEMIJOIN`：启用或禁用指定的半连接策略。

+   *`strategy`*：要启用或禁用的半连接策略。允许使用这些策略名称：`DUPSWEEDOUT`、`FIRSTMATCH`、`LOOSESCAN`、`MATERIALIZATION`。

    对于`SEMIJOIN`提示，如果没有指定策略，则根据根据`optimizer_switch`系统变量启用的策略来尽可能使用半连接。如果指定了策略但对语句不适用，则使用`DUPSWEEDOUT`。

    对于`NO_SEMIJOIN`提示，如果没有指定策略，则不使用半连接。如果指定的策略排除了语句的所有适用策略，则使用`DUPSWEEDOUT`。

如果一个子查询嵌套在另一个子查询中，并且两者合并为外部查询的半连接，那么对于最内层查询的任何半连接策略规范都将被忽略。仍然可以使用`SEMIJOIN`和`NO_SEMIJOIN`提示来启用或禁用这种嵌套子查询的半连接转换。

如果禁用了`DUPSWEEDOUT`，有时优化器可能生成远非最佳的查询计划。这是由于贪婪搜索期间的启发式修剪导致的，可以通过设置`optimizer_prune_level=0`来避免。

示例：

```sql
SELECT /*+ NO_SEMIJOIN(@subq1 FIRSTMATCH, LOOSESCAN) */ * FROM t2
  WHERE t2.a IN (SELECT /*+ QB_NAME(subq1) */ a FROM t3);
SELECT /*+ SEMIJOIN(@subq1 MATERIALIZATION, DUPSWEEDOUT) */ * FROM t2
  WHERE t2.a IN (SELECT /*+ QB_NAME(subq1) */ a FROM t3);
```

影响是否使用子查询材料化或`IN`到`EXISTS`转换的提示的语法：

```sql
SUBQUERY([@*query_block_name*] *strategy*)
```

提示名称始终为`SUBQUERY`。

对于`SUBQUERY`提示，允许使用这些*`strategy`*值：`INTOEXISTS`，`MATERIALIZATION`。

例子：

```sql
SELECT id, a IN (SELECT /*+ SUBQUERY(MATERIALIZATION) */ a FROM t1) FROM t2;
SELECT * FROM t2 WHERE t2.a IN (SELECT /*+ SUBQUERY(INTOEXISTS) */ a FROM t1);
```

对于半连接和`SUBQUERY`提示，前导`@*`query_block_name`*`指定提示适用的查询块。如果提示不包含前导`@*`query_block_name`*`，则提示适用于其出现的查询块。要为查询块分配名称，请参阅用于命名查询块的优化提示。

如果提示注释包含多个子查询提示，则使用第一个。如果有其他相同类型的后续提示，则会产生警告。其他类型的后续提示会被静默忽略。

#### 语句执行时间优化提示

`MAX_EXECUTION_TIME`提示仅适用于`SELECT`语句。它在服务器终止之前对语句执行的时间设置了限制*`N`*（以毫秒为单位的超时值）：

```sql
MAX_EXECUTION_TIME(*N*)
```

例如，设置超时为 1 秒（1000 毫秒）：

```sql
SELECT /*+ MAX_EXECUTION_TIME(1000) */ * FROM t1 INNER JOIN t2 WHERE ...
```

`MAX_EXECUTION_TIME(*`N`*)`提示设置了*`N`*毫秒的语句执行超时。如果此选项不存在或*`N`*为 0，则由`max_execution_time`系统变量设置的语句超时生效。

`MAX_EXECUTION_TIME`提示适用如下：

+   对于包含多个`SELECT`关键字的语句，例如联合或包含子查询的语句，`MAX_EXECUTION_TIME`适用于整个语句，并且必须出现在第一个`SELECT`之后。

+   适用于只读`SELECT`语句。不是只读的语句是那些调用修改数据的存储函数的语句。

+   不适用于存储程序中的`SELECT`语句，并且会被忽略。

#### 变量设置提示语法

`SET_VAR`提示临时设置系统变量的会话值（在单个语句的持续时间内）。例子：

```sql
SELECT /*+ SET_VAR(sort_buffer_size = 16M) */ name FROM people ORDER BY name;
INSERT /*+ SET_VAR(foreign_key_checks=OFF) */ INTO t2 VALUES(2);
SELECT /*+ SET_VAR(optimizer_switch = 'mrr_cost_based=off') */ 1;
```

`SET_VAR`提示的语法：

```sql
SET_VAR(*var_name* = *value*)
```

*`var_name`* 是命名具有会话值的系统变量（尽管并非所有这些变量都可以命名，稍后会解释）。*`value`* 是要分配给变量的值；该值必须是标量。

`SET_VAR` 进行临时变量更改，如下面的语句所示：

```sql
mysql> SELECT @@unique_checks;
+-----------------+
| @@unique_checks |
+-----------------+
|               1 |
+-----------------+
mysql> SELECT /*+ SET_VAR(unique_checks=OFF) */ @@unique_checks;
+-----------------+
| @@unique_checks |
+-----------------+
|               0 |
+-----------------+
mysql> SELECT @@unique_checks;
+-----------------+
| @@unique_checks |
+-----------------+
|               1 |
+-----------------+
```

使用 `SET_VAR`，无需保存和恢复变量值。这使您可以通过单个语句替换多个语句。考虑以下语句序列：

```sql
SET @saved_val = @@SESSION.*var_name*;
SET @@SESSION.*var_name* = *value*;
SELECT ...
SET @@SESSION.*var_name* = @saved_val;
```

这个序列可以被这个单个语句替代：

```sql
SELECT /*+ SET_VAR(*var_name* = *value*) ...
```

独立的 `SET` 语句允许使用以下任何语法来命名会话变量：

```sql
SET SESSION *var_name* = *value*;
SET @@SESSION.*var_name* = *value*;
SET @@.*var_name* = *value*;
```

因为 `SET_VAR` 提示仅适用于会话变量，会话范围是隐含的，`SESSION`、`@@SESSION.` 和 `@@` 既不需要也不允许。包括显式会话指示符语法会导致忽略带有警告的 `SET_VAR` 提示。

并非所有会话变量都可以与 `SET_VAR` 一起使用。各个系统变量描述指示每个变量是否可提示；请参阅 第 7.1.8 节，“服务器系统变量”。您还可以通过尝试在 `SET_VAR` 中使用它来在运行时检查系统变量。如果变量不可提示，则会发出警告：

```sql
mysql> SELECT /*+ SET_VAR(collation_server = 'utf8mb4') */ 1;
+---+
| 1 |
+---+
| 1 |
+---+
1 row in set, 1 warning (0.00 sec)

mysql> SHOW WARNINGS\G
*************************** 1\. row ***************************
  Level: Warning
   Code: 4537
Message: Variable 'collation_server' cannot be set using SET_VAR hint.
```

`SET_VAR` 语法允许设置单个变量，但可以提供多个提示以设置多个变量：

```sql
SELECT /*+ SET_VAR(optimizer_switch = 'mrr_cost_based=off')
           SET_VAR(max_heap_table_size = 1G) */ 1;
```

如果在同一语句中出现多个具有相同变量名称的提���，第一个将被应用，其他的将被忽略并显示警告：

```sql
SELECT /*+ SET_VAR(max_heap_table_size = 1G)
           SET_VAR(max_heap_table_size = 3G) */ 1;
```

在这种情况下，第二个提示将被忽略并显示冲突警告。

如果没有系统变量具有指定名称或变量值不正确，则会忽略带有警告的 `SET_VAR` 提示：

```sql
SELECT /*+ SET_VAR(max_size = 1G) */ 1;
SELECT /*+ SET_VAR(optimizer_switch = 'mrr_cost_based=yes') */ 1;
```

对于第一条语句，不存在 `max_size` 变量。对于第二条语句，`mrr_cost_based` 可以取值 `on` 或 `off`，因此尝试将其设置为 `yes` 是不正确的。在每种情况下，提示都会被忽略并显示警告。

`SET_VAR` 提示仅在语句级别允许。如果在子查询中使用，提示将被忽略并显示警告。

复制品会忽略复制语句中的 `SET_VAR` 提示，以避免安全问题的潜在风险。

#### 资源组提示语法

`RESOURCE_GROUP` 优化提示用于资源组管理（参见第 7.1.16 节，“资源组”）。此提示将执行语句的线程暂时分配给命名的资源组（在语句执行期间）。它需要`RESOURCE_GROUP_ADMIN` 或 `RESOURCE_GROUP_USER` 权限。

例子：

```sql
SELECT /*+ RESOURCE_GROUP(USR_default) */ name FROM people ORDER BY name;
INSERT /*+ RESOURCE_GROUP(Batch) */ INTO t2 VALUES(2);
```

`RESOURCE_GROUP` 提示的语法：

```sql
RESOURCE_GROUP(*group_name*)
```

*`group_name`* 表示线程在语句执行期间应分配给的资源组。如果组不存在，则会发出警告并忽略提示。

`RESOURCE_GROUP` 提示必须出现在初始语句关键字（`SELECT`、`INSERT`、`REPLACE`、`UPDATE` 或 `DELETE`）之后。

与 `RESOURCE_GROUP` 的另一种选择是 `SET RESOURCE GROUP` 语句，它将线程非暂时地分配给资源组。参见第 15.7.2.4 节，“SET RESOURCE GROUP 语句”。

#### 为查询块命名的优化提示

表级、索引级和子查询优化提示允许特定查询块作为其参数语法的一部分命名。要创建这些名称，请使用 `QB_NAME` 提示，它为其出现的查询块分配一个名称：

```sql
QB_NAME(*name*)
```

`QB_NAME` 可以清晰地表明其他提示适用于哪些查询块。它们还允许在单个提示注释中指定所有非查询块名称提示，以便更容易理解复杂语句。考虑以下语句：

```sql
SELECT ...
  FROM (SELECT ...
  FROM (SELECT ... FROM ...)) ...
```

`QB_NAME` 提示为语句中的查询块分配名称：

```sql
SELECT /*+ QB_NAME(qb1) */ ...
  FROM (SELECT /*+ QB_NAME(qb2) */ ...
  FROM (SELECT /*+ QB_NAME(qb3) */ ... FROM ...)) ...
```

然后其他提示可以使用这些名称来引用适当的查询块：

```sql
SELECT /*+ QB_NAME(qb1) MRR(@qb1 t1) BKA(@qb2) NO_MRR(@qb3t1 idx1, id2) */ ...
  FROM (SELECT /*+ QB_NAME(qb2) */ ...
  FROM (SELECT /*+ QB_NAME(qb3) */ ... FROM ...)) ...
```

结果效果如下：

+   `MRR(@qb1 t1)` 适用于查询块 `qb1` 中的表 `t1`。

+   `BKA(@qb2)` 适用于查询块 `qb2`。

+   `NO_MRR(@qb3 t1 idx1, id2)` 适用于查询块 `qb3` 中表 `t1` 中的索引 `idx1` 和 `idx2`。

查询块名称是标识符，并遵循关于名称有效性和如何引用它们的通常规则（参见第 11.2 节，“模式对象名称”）。例如，包含空格的查询块名称必须用引号引起来，可以使用反引号：

```sql
SELECT /*+ BKA(@`my hint name`) */ ...
  FROM (SELECT /*+ QB_NAME(`my hint name`) */ ...) ...
```

如果启用了`ANSI_QUOTES` SQL 模式，则还可以用双引号引用查询块名称：

```sql
SELECT /*+ BKA(@"my hint name") */ ...
  FROM (SELECT /*+ QB_NAME("my hint name") */ ...) ...
```
