# 10.8.2 EXPLAIN 输出格式

> 原文：[`dev.mysql.com/doc/refman/8.0/en/explain-output.html`](https://dev.mysql.com/doc/refman/8.0/en/explain-output.html)

`EXPLAIN`语句提供有关 MySQL 如何执行语句的信息。`EXPLAIN`与`SELECT`，`DELETE`，`INSERT`，`REPLACE`和`UPDATE`语句一起使用。

`EXPLAIN`为`SELECT`语句中使用的每个表返回一行信息。它按照 MySQL 在处理语句时读取它们的顺序在输出中列出表。这意味着 MySQL 从第一个表中读取一行，然后在第二个表中找到匹配的行，然后在第三个表中找到匹配的行，依此类推。当所有表都被处理时，MySQL 输出所选列，并通过表列表回溯，直到找到一个表，其中有更多匹配的行。从这个表中读取下一行，然后继续下一个表的过程。

注意

MySQL Workbench 具有 Visual Explain 功能，提供了`EXPLAIN`输出的可视化表示。请参阅 Tutorial: Using Explain to Improve Query Performance。

+   EXPLAIN 输出列

+   EXPLAIN 连接类型

+   EXPLAIN 额外信息

+   EXPLAIN 输出解释

#### EXPLAIN 输出列

本节描述了`EXPLAIN`生成的输出列。后续章节提供有关`type`和`Extra`列的附加信息。

每个`EXPLAIN`的输出行提供有关一个表的信息。每行包含表 10.1，“EXPLAIN 输出列”中总结的值，并在表后详细描述。表中显示列名在表的第一列；第二列提供了在使用`FORMAT=JSON`时输出中显示的等效属性名称。

**表 10.1 EXPLAIN 输出列**

| 列 | JSON 名称 | 含义 |
| --- | --- | --- |
| `id` | `select_id` | `SELECT`标识符 |
| `select_type` | None | `SELECT` 类型 |
| `table` | `table_name` | 输出行的表 |
| `partitions` | `partitions` | 匹配的分区 |
| `type` | `access_type` | 连接类型 |
| `possible_keys` | `possible_keys` | 可选择的索引 |
| `key` | `key` | 实际选择的索引 |
| `key_len` | `key_length` | 所选键的长度 |
| `ref` | `ref` | 与索引进行比较的列 |
| `rows` | `rows` | 预计要检查的行数 |
| `filtered` | `filtered` | 表条件过滤的行的百分比 |
| `Extra` | None | 附加信息 |
| 列 | JSON 名称 | 含义 |

注意

在 JSON 格式的 `EXPLAIN` 输出中，值为 `NULL` 的 JSON 属性不会显示。

+   `id`（JSON 名称：`select_id`）

    `SELECT`标识符。这是查询中`SELECT`的顺序号。如果行引用其他行的联合结果，则该值可以为`NULL`。在这种情况下，`table`列显示类似`<union*`M`*,*`N`*>`的值，表示该行引用具有`id`值为*`M`*和*`N`*的行的联合。

+   `select_type`（JSON 名称：无）

    `SELECT`的类型，可以是以下表中显示的任何类型之一。JSON 格式的 `EXPLAIN` 将 `SELECT` 类型公开为 `query_block` 的属性，除非它是 `SIMPLE` 或 `PRIMARY`。表中还显示了 JSON 名称（如果适用）。

    | `select_type` 值 | JSON 名称 | 含义 |
    | --- | --- | --- |
    | `SIMPLE` | None | 简单的`SELECT`（不使用`UNION`或子查询） |
    | `PRIMARY` | None | 最外层的`SELECT` |
    | `UNION` | None | `UNION`中的第二个或更后的`SELECT`语句 |
    | `DEPENDENT UNION` | `dependent` (`true`) | `UNION`中的第二个或更后的`SELECT`语句，依赖于外部查询 |
    | `UNION RESULT` | `union_result` | `UNION`的结果。 |
    | `SUBQUERY` | None | 子查询中的第一个`SELECT` |
    | `DEPENDENT SUBQUERY` | `dependent`（`true`） | 子查询中的第一个`SELECT`，依赖于外部查询 |
    | `DERIVED` | 无 | 派生表 |
    | `DEPENDENT DERIVED` | `dependent`（`true`） | 依赖于另一张表的派生表 |
    | `MATERIALIZED` | `materialized_from_subquery` | 物化子查询 |
    | `UNCACHEABLE SUBQUERY` | `cacheable`（`false`） | 不能缓存结果且必须对外部查询的每一行重新评估的子查询 |
    | `UNCACHEABLE UNION` | `cacheable`（`false`） | `UNION`中的第二个或更后续的选择属于不可缓存子查询（参见`UNCACHEABLE SUBQUERY`） |
    | `select_type`值 | JSON 名称 | 含义 |

    `DEPENDENT`通常表示使用相关子查询。参见第 15.2.15.7 节，“相关子查询”。

    `DEPENDENT SUBQUERY`评估与`UNCACHEABLE SUBQUERY`评估不同。对于`DEPENDENT SUBQUERY`，子查询仅针对其外部上下文的不同变量值集重新评估一次。对于`UNCACHEABLE SUBQUERY`，子查询对外部上下文的每一行重新评估。

    当你在`EXPLAIN`中指定`FORMAT=JSON`时，输出中没有与`select_type`直接等价的单个属性；`query_block`属性对应于给定的`SELECT`。大多数刚刚显示的`SELECT`子查询类型的等效属性是可用的（例如`materialized_from_subquery`对应于`MATERIALIZED`），并在适当时显示。对于`SIMPLE`或`PRIMARY`，没有 JSON 等价物。

    非`SELECT`语句的`select_type`值显示受影响表的语句类型。例如，对于`DELETE`语句，`select_type`为`DELETE`。

+   `table`（JSON 名称：`table_name`）

    输出行所指的表的名称。这也可以是以下值之一：

    +   `<union*`M`*,*`N`*>`：该行指的是具有`id`值为*`M`*和*`N`*的行的并集。

    +   `<derived*`N`*>`：该行指的是具有`id`值为*`N`*的行的派生表结果。派生表可能是由`FROM`子句中的子查询导致的。

    +   `<subquery*`N`*>`：该行指的是具有`id`值为*`N`*的行的物化子查询的结果。参见第 10.2.2.2 节，“使用物化进行子查询优化”。

+   `partitions`（JSON 名称：`partitions`）

    查询将匹配的分区。对于非分区表，该值为`NULL`。参见第 26.3.5 节，“获取有关分区的信息”。

+   `type`（JSON 名称：`access_type`）

    连接类型。有关不同类型的描述，请参见`EXPLAIN` Join Types。

+   `possible_keys`（JSON 名称：`possible_keys`）

    `possible_keys`列指示 MySQL 可以选择从中查找此表中的行的索引。请注意，此列与从`EXPLAIN`输出中显示的表的顺序无关。这意味着`possible_keys`中的一些键在实践中可能无法与生成的表顺序一起使用。

    如果此列为`NULL`（或在 JSON 格式输出中未定义），则没有相关的索引。在这种情况下，您可以通过检查`WHERE`子句来查看是否引用了适合索引的某些列，从而改善查询的性能。如果是这样，请创建一个适当的索引，并再次使用`EXPLAIN`检查查询。参见第 15.1.9 节，“ALTER TABLE Statement”。

    要查看表具有哪些索引，请使用`SHOW INDEX FROM *`tbl_name`*`。

+   `key`（JSON 名称：`key`）

    `key`列指示 MySQL 实际决定使用的键（索引）。如果 MySQL 决定使用`possible_keys`中的一个索引来查找行，那么该索引将列为键值。

    `key`可能指的是`possible_keys`值中不存在的索引。如果`possible_keys`中的索引都不适合查找行，但查询选择的所有列都是某个其他索引的列，就会发生这种情况。也就是说，命名的索引覆盖了选择的列，因此虽然它不用于确定要检索哪些行，但索引扫描比数据行扫描更有效率。

    对于`InnoDB`，即使查询还选择了主键，次要索引也可能覆盖了选择的列，因为`InnoDB`将主键值与每个次要索引一起存储。如果`key`为`NULL`，则 MySQL 找不到用于更有效地执行查询的索引。

    要强制 MySQL 使用或忽略`possible_keys`列中列出的索引，请在查询中使用`FORCE INDEX`、`USE INDEX`或`IGNORE INDEX`。参见第 10.9.4 节，“Index Hints”。

    对于`MyISAM`表，运行`ANALYZE TABLE`有助于优化器选择更好的索引。对于`MyISAM`表，**myisamchk --analyze**也是一样的。参见第 15.7.3.1 节，“ANALYZE TABLE Statement”，以及第 9.6 节，“MyISAM Table Maintenance and Crash Recovery”。

+   `key_len`（JSON 名称：`key_length`）

    `key_len`列指示 MySQL 决定使用的键的长度。`key_len`的值使您能够确定 MySQL 实际使用多部分键的部分数。如果`key`列说`NULL`，则`key_len`列也说`NULL`。

    由于键存储格式，对于可以为`NULL`的列，键长度比`NOT NULL`列多一个。

+   `ref`（JSON 名称：`ref`）

    `ref`列显示与在`key`列中命名的索引进行比较以从表中选择行的哪些列或常量。

    如果值为`func`，则使用的值是某个函数的结果。要查看哪个函数，请在`EXPLAIN`之后使用`SHOW WARNINGS`查看扩展的`EXPLAIN`输出。该函数实际上可能是算术运算符等运算符。

+   `rows`（JSON 名称：`rows`）

    `rows`列指示 MySQL 认为必须检查的行数以执行查询。

    对于`InnoDB`表，此数字是一个估计值，可能并不总是准确的。

+   `filtered`（JSON 名称：`filtered`）

    `filtered`列指示由表条件过滤的表行的估计百分比。最大值为 100，表示未发生任何行过滤。从 100 递减的值表示过滤量增加。`rows`显示估计检查的行数，`rows` × `filtered`显示与下表连接的行数。例如，如果`rows`为 1000，`filtered`为 50.00（50％），则要与下表连接的行数为 1000 × 50％= 500。

+   `Extra`（JSON 名称：无）

    此列包含有关 MySQL 如何解析查询的其他信息。有关不同值的描述，请参见`EXPLAIN` Extra Information。

    没有与`Extra`列对应的单个 JSON 属性；但是，此列中可能出现的值会作为 JSON 属性或作为`message`属性的文本公开。

#### 解释连接类型

`EXPLAIN`输出的`type`列描述了表是如何连接的。在 JSON 格式的输出中，这些被发现为`access_type`属性的值。以下列表描述了连接类型，从最佳类型到最差类型：

+   `system`

    表只有一行（=系统表）。这是`const`连接类型的特殊情况。

+   `const`

    该表最多只有一行匹配行，在查询开始时读取。因为只有一行，来自此行的列值可以被优化器的其余部分视为常量。`const`表非常快，因为只读取一次。

    当您将`PRIMARY KEY`或`UNIQUE`索引的所有部分与常量值进行比较时，将使用`const`。在以下查询中，*`tbl_name`*可以作为`const`表使用：

    ```sql
    SELECT * FROM *tbl_name* WHERE *primary_key*=1;

    SELECT * FROM *tbl_name*
      WHERE *primary_key_part1*=1 AND *primary_key_part2*=2;
    ```

+   `eq_ref`

    对于前面表的每一行组合，从该表中读取一行。除了`system`和`const`类型之外，这是最佳的连接类型。当连接使用索引的所有部分并且索引是`PRIMARY KEY`或`UNIQUE NOT NULL`索引时使用。

    可以用于使用`=`运算符比较的索引列。比较值可以是常量或使用在此表之前读取的表的列的表达式。在以下示例中，MySQL 可以使用`eq_ref`连接来处理*`ref_table`*：

    ```sql
    SELECT * FROM *ref_table*,*other_table*
      WHERE *ref_table*.*key_column*=*other_table*.*column*;

    SELECT * FROM *ref_table*,*other_table*
      WHERE *ref_table*.*key_column_part1*=*other_table*.*column*
      AND *ref_table*.*key_column_part2*=1;
    ```

+   `ref`

    对于前面表的每一行组合，从该表中读取具有匹配索引值的所有行。如果连接仅使用键的最左前缀或键不是`PRIMARY KEY`或`UNIQUE`索引（换句话说，如果连接不能基于键值选择单行），则使用`ref`。如果使用的键仅匹配少数行，则这是一种很好的连接类型。

    可以用于使用`=`或`<=>`运算符比较的索引列。在以下示例中，MySQL 可以使用`ref`连接来处理*`ref_table`*：

    ```sql
    SELECT * FROM *ref_table* WHERE *key_column*=*expr*;

    SELECT * FROM *ref_table*,*other_table*
      WHERE *ref_table*.*key_column*=*other_table*.*column*;

    SELECT * FROM *ref_table*,*other_table*
      WHERE *ref_table*.*key_column_part1*=*other_table*.*column*
      AND *ref_table*.*key_column_part2*=1;
    ```

+   `fulltext`

    使用`FULLTEXT`索引执行连接。

+   `ref_or_null`

    这种连接类型类似于`ref`，但额外搜索包含`NULL`值的行。这种连接类型优化通常在解析子查询时最常使用。在以下示例中，MySQL 可以使用`ref_or_null`连接来处理*`ref_table`*：

    ```sql
    SELECT * FROM *ref_table*
      WHERE *key_column*=*expr* OR *key_column* IS NULL;
    ```

    请参阅第 10.2.1.15 节，“IS NULL Optimization”。

+   `index_merge`

    此连接类型指示使用了索引合并优化。在这种情况下，输出行中的`key`列包含使用的索引列表，`key_len`包含用于索引的最长键部分列表。有关更多信息，请参见第 10.2.1.3 节，“索引合并优化”。

+   `unique_subquery`

    对于以下形式的一些`IN`子查询，此类型替换了`eq_ref`：

    ```sql
    *value* IN (SELECT *primary_key* FROM *single_table* WHERE *some_expr*)
    ```

    `unique_subquery`只是一个索引查找函数，完全替换子查询以提高效率。

+   `index_subquery`

    此连接类型类似于`unique_subquery`。它替换了`IN`子查询，但适用于以下形式的非唯一索引子查询：

    ```sql
    *value* IN (SELECT *key_column* FROM *single_table* WHERE *some_expr*)
    ```

+   `range`

    仅检索给定范围内的行，使用索引选择行。输出行中的`key`列指示使用的索引。`key_len`包含使用的最长键部分。对于这种类型，`ref`列为`NULL`。

    当使用任何`=`、`<>`、`>`、`>=`、`<`、`<=`、`IS NULL`、`<=>`、`BETWEEN`、`LIKE`或`IN()`运算符将键列与常量进行比较时，可以使用`range`：

    ```sql
    SELECT * FROM *tbl_name*
      WHERE *key_column* = 10;

    SELECT * FROM *tbl_name*
      WHERE *key_column* BETWEEN 10 and 20;

    SELECT * FROM *tbl_name*
      WHERE *key_column* IN (10,20,30);

    SELECT * FROM *tbl_name*
      WHERE *key_part1* = 10 AND *key_part2* IN (10,20,30);
    ```

+   `index`

    `index`连接类型与`ALL`相同，只是扫描索引树。有两种情况：

    +   如果索引是查询的覆盖索引，并且可以用于满足表中所需的所有数据，那么只扫描索引树。在这种情况下，`Extra`列显示`Using index`。索引扫描通常比`ALL`更快，因为索引的大小通常比表数据小。

    +   执行全表扫描，使用从索引读取的数据行按索引顺序查找数据行。`Extra`列中不会出现`Uses index`。

    当查询仅使用属于单个索引的列时，MySQL 可以使用此连接类型。

+   `ALL`

    对于前面表的每个行组合，都会进行完整表扫描。如果表不是第一个未标记为 `const` 的表，通常情况下这是不好的，而在其他情况下通常是*非常*糟糕的。通常，您可以通过添加索引来避免 `ALL`，这些索引使得可以基于常量值或来自早期表的列值检索行。

#### EXPLAIN 额外信息

`EXPLAIN` 输出的 `Extra` 列包含关于 MySQL 如何解析查询的附加信息。以下列表解释了此列中可能出现的值。每个项目还指示了 JSON 格式输出中显示 `Extra` 值的属性。对于其中一些，有一个特定的属性。其他显示为 `message` 属性的文本。

如果要使查询尽可能快速，请注意 `Extra` 列值为 `Using filesort` 和 `Using temporary`，或者在 JSON 格式的 `EXPLAIN` 输出中，`using_filesort` 和 `using_temporary_table` 属性等于 `true`。

+   `Backward index scan` (JSON：`backward_index_scan`)

    优化器能够在 `InnoDB` 表上使用降序索引。与 `Using index` 一起显示。有关更多信息，请参见 Section 10.3.13, “Descending Indexes”。

+   `Child of '*`table`*' pushed join@1` (JSON：`message` 文本)

    此表被引用为 *`table`* 的子表，可以将其推送到 NDB 内核中进行连接。仅适用于 NDB Cluster，在启用推送连接时。有关更多信息和示例，请参见 `ndb_join_pushdown` 服务器系统变量的描述。

+   `const row not found` (JSON 属性：`const_row_not_found`)

    对于诸如 `SELECT ... FROM *`tbl_name`*` 的查询，表是空的。

+   `Deleting all rows` (JSON 属性：`message`)

    对于 `DELETE`，一些存储引擎（例如 `MyISAM`）支持一种移除所有表行的处理程序方法，以简单快速的方式执行。如果引擎使用此优化，则显示此 `Extra` 值。

+   `Distinct` (JSON 属性：`distinct`)

    MySQL 正在寻找不同的值，因此在找到第一个匹配行后，停止为当前行组合搜索更多行。

+   `FirstMatch(*`tbl_name`*)` (JSON 属性：`first_match`)

    半连接 FirstMatch 加入快捷策略用于 *`tbl_name`*。

+   `Full scan on NULL key` (JSON 属性：`message`)

    当优化器无法使用索引查找访问方法时，对于子查询优化，这是一个后备策略。

+   `Impossible HAVING` (JSON 属性：`message`)

    `HAVING` 子句始终为假，无法选择任何行。

+   `Impossible WHERE` (JSON 属性：`message`)

    `WHERE`子句始终为 false，无法选择任何行。

+   `在读取 const 表后注意到不可能的 WHERE`（JSON 属性：`message`）。

    MySQL 已读取所有`const`（和`system`）表，并注意到`WHERE`子句始终为 false。

+   `LooseScan(*`m`*..*`n`*)`（JSON 属性：`message`）。

    使用半连接 LooseScan 策略。*`m`*和*`n`*是关键部分号。

+   `没有匹配的最小/最大行`（JSON 属性：`message`）。

    没有任何行满足查询条件，比如`SELECT MIN(...) FROM ... WHERE *`condition`*`。

+   `const`表中没有匹配的行（JSON 属性：`message`）。

    对于具有连接的查询，存在一个空表或一个表不满足唯一索引条件的行。

+   `分区修剪后没有匹配的行`（JSON 属性：`message`）。

    对于`DELETE`或`UPDATE`语句，在分区修剪后，优化器发现没有要删除或更新的内容。这在某种程度上类似于`SELECT`语句的`Impossible WHERE`。

+   `未使用任何表`（JSON 属性：`message`）。

    查询没有`FROM`子句，或者有一个`FROM DUAL`子句。

    对于`INSERT`或`REPLACE`语句，当没有`SELECT`部分时，`EXPLAIN`显示此值。例如，对于`EXPLAIN INSERT INTO t VALUES(10)`，因为这等效于`EXPLAIN INSERT INTO t SELECT 10 FROM DUAL`。

+   `不存在`（JSON 属性：`message`）。

    MySQL 能够在查询上进行`LEFT JOIN`优化，并在找到符合`LEFT JOIN`条件的一行后，不再检查此表中的其他行组合。以下是可以通过这种方式优化的查询类型示例：

    ```sql
    SELECT * FROM t1 LEFT JOIN t2 ON t1.id=t2.id
      WHERE t2.id IS NULL;
    ```

    假设`t2.id`被定义为`NOT NULL`。在这种情况下，MySQL 扫描`t1`并使用`t1.id`的值查找`t2`中的行。如果 MySQL 在`t2`中找到匹配的行，它知道`t2.id`永远不会是`NULL`，并且不会扫描具有相同`id`值的`t2`中的其他行。换句话说，对于`t1`中的每一行，MySQL 只需要在`t2`中进行一次查找，而不管实际上有多少行与`t2`中的`id`值匹配。

    在 MySQL 8.0.17 及更高版本中，这也可能表示形如`NOT IN (*`subquery`*)`或`NOT EXISTS (*`subquery`*)`的`WHERE`条件已在内部转换为反连接。这会删除子查询，并将其表合并到顶层查询的计划中，提供了更好的成本规划。通过合并半连接和反连接，优化器可以更自由地重新排列执行计划中的表，在某些情况下可以得到更快的计划。

    通过在执行`EXPLAIN`后检查`SHOW WARNINGS`中的`Message`列，或在`EXPLAIN FORMAT=TREE`的输出中，可以看到给定查询执行时进行的反连接转换。

    注意

    反连接是半连接`*`table_a`* JOIN *`table_b`* ON *`condition`*`的补集。反连接返回*`table_a`*中所有没有与*`table_b`*匹配*`condition`*的行。

+   `计划尚未准备好`（JSON 属性：无）

    当优化器尚未完成为在命名连接中执行的语句创建执行计划时，会出现`EXPLAIN FOR CONNECTION`中的这个值。如果执行计划输出包含多行，则根据优化器在确定完整执行计划的进度而定，任何一行或所有行都可能具有这个`Extra`值。

+   `Range checked for each record (index map: *`N`*)`（JSON 属性：`message`）

    MySQL 没有找到合适的索引可用，但发现在了解前面表的列值后，可能会使用一些索引。对于前面表中的每一行组合，MySQL 检查是否可以使用`range`或`index_merge`访问方法来检索行。这并不是非常快速，但比完全没有索引的连接要快。适用条件如第 10.2.1.2 节，“范围优化”和第 10.2.1.3 节，“索引合并优化”中所述，唯一的例外是前面表的所有列值都已知且被视为常量。

    索引从 1 开始编号，与表的`SHOW INDEX`显示的顺序相同。索引映射值*`N`*是一个位掩码值，指示哪些索引是候选索引。例如，值`0x19`（二进制 11001）表示索引 1、4 和 5 被考虑。

+   `Recursive`（JSON 属性：`recursive`）

    这表示该行适用于递归`SELECT`的部分，即递归公共表达式。参见第 15.2.20 节，“WITH（公共表达式）”。

+   `Rematerialize`（JSON 属性：`rematerialize`）

    在表`T`的`EXPLAIN`行中显示`Rematerialize (X,...)`，其中`X`是任何侧向派生表，当读取`T`的新行时触发其重新生成。例如：

    ```sql
    SELECT
      ...
    FROM
      t,
      LATERAL (*derived table that refers to t*) AS dt
    ...
    ```

    派生表的内容在每次顶部查询处理新行时重新生成，以使其保持最新状态。

+   `扫描了*`N`*个数据库`（JSON 属性：`message`）

    这指示服务器在处理`INFORMATION_SCHEMA`表查询时执行多少目录扫描，如 Section 10.2.3, “Optimizing INFORMATION_SCHEMA Queries”中所述。*`N`*的值可以是 0、1 或`all`。

+   `Select tables optimized away`（JSON 属性：`message`）

    优化器确定了以下两点：1）最多只返回一行，2）为了产生这一行，必须读取一组确定性的行。当要读取的行可以在优化阶段读取时（例如，通过读取索引行），在查询执行期间就不需要读取任何表。

    当查询隐式分组（包含聚合函数但没有`GROUP BY`子句）时，第一个条件得到满足。当每个使用的索引执行一次行查找时，第二个条件得到满足。读取的索引数决定了要读取的行数。

    考虑以下隐式分组查询：

    ```sql
    SELECT MIN(c1), MIN(c2) FROM t1;
    ```

    假设通过读取一个索引行可以检索到`MIN(c1)`，通过从不同索引读取一行可以检索到`MIN(c2)`。也就是说，对于每个列`c1`和`c2`，存在一个索引，其中该列是索引的第一列。在这种情况下，通过读取两个确定性行生成一行返回。

    如果要读取的行不是确定性的，则不会出现此`Extra`值。考虑以下查询：

    ```sql
    SELECT MIN(c2) FROM t1 WHERE c1 <= 10;
    ```

    假设`(c1, c2)`是一个覆盖索引。使用此索引，必须扫描所有`c1 <= 10`的行以找到最小的`c2`值。相比之下，考虑以下查询：

    ```sql
    SELECT MIN(c2) FROM t1 WHERE c1 = 10;
    ```

    在这种情况下，具有`c1 = 10`的第一个索引行包含最小的`c2`值。只需读取一行即可生成返回的行。

    对于维护每个表的确切行数的存储引擎（例如`MyISAM`，但不包括`InnoDB`），对于缺少或始终为真的`WHERE`子句且没有`GROUP BY`子句的`COUNT(*)`查询，此`Extra`值可能出现。（这是存储引擎影响是否可以读取确定数量的行的隐式分组查询的一个实例。）

+   `Skip_open_table`，`Open_frm_only`，`Open_full_table`（JSON 属性：`message`）

    这些值指示适用于`INFORMATION_SCHEMA`表查询的文件打开优化。

    +   `Skip_open_table`：不需要打开表文件。信息已经从数据字典中获取。

    +   `Open_frm_only`：只需读取数据字典以获取表信息。

    +   `Open_full_table`：未优化的信息查找。必须从数据字典中读取表信息，并通过读取表文件。

+   `Start temporary`，`End temporary`（JSON 属性：`message`）

    这表明临时表用于半连接去重策略。

+   `unique row not found`（JSON 属性：`message`）

    对于诸如`SELECT ... FROM *tbl_name*`的查询，没有行满足表上的`UNIQUE`索引或`PRIMARY KEY`的条件。

+   `Using filesort`（JSON 属性：`using_filesort`）

    MySQL 必须进行额外的传递来找出如何按排序顺序检索行。排序是通过根据连接类型遍历所有行并为匹配`WHERE`子句的所有行存储排序键和指向行的指针来完成的。然后对键进行排序，并按排序顺序检索行。请参阅第 10.2.1.16 节，“ORDER BY 优化”。

+   `Using index`（JSON 属性：`using_index`）

    仅使用索引树中的信息从表中检索列信息，而无需进行额外的查找以读取实际行。当查询仅使用单个索引的列时，可以使用此策略。

    对于具有用户定义的聚簇索引的`InnoDB`表，即使`Extra`列中缺少`Using index`，该索引也可以使用。如果`type`是`index`，并且`key`是`PRIMARY`，则是这种情况。

    有关使用的任何覆盖索引的信息显示在`EXPLAIN FORMAT=TRADITIONAL`和`EXPLAIN FORMAT=JSON`中。从 MySQL 8.0.27 开始，也会显示在`EXPLAIN FORMAT=TREE`中。

+   `Using index condition`（JSON 属性：`using_index_condition`）

    通过访问索引元组并首先测试它们来读取表。通过这种方式，索引信息用于推迟（“推下去”）读取完整的表行，除非有必要。请参阅第 10.2.1.6 节，“索引条件推迟优化”。

+   `Using index for group-by`（JSON 属性：`using_index_for_group_by`）

    类似于`Using index`表访问方法，`Using index for group-by`表示 MySQL 找到了一个可以用于检索`GROUP BY`或`DISTINCT`查询的所有列的索引，而无需额外访问实际表的磁盘。此外，索引以最有效的方式使用，以便对于每个组，只读取了少量索引条目。有关详细信息，请参阅第 10.2.1.17 节，“GROUP BY 优化”。

+   `Using index for skip scan`（JSON 属性：`using_index_for_skip_scan`)

    表示使用了 Skip Scan 访问方法。请参阅 Skip Scan 范围访问方法。

+   `Using join buffer (Block Nested Loop)`，`Using join buffer (Batched Key Access)`，`Using join buffer (hash join)`（JSON 属性：`using_join_buffer`）

    早期连接的表被分段读入连接缓冲区，然后它们的行从缓冲区中使用来与当前表进行连接。`(Block Nested Loop)`表示使用块嵌套循环算法，`(Batched Key Access)`表示使用批量键访问算法，`(hash join)`表示使用哈希连接。也就是说，前一行`EXPLAIN`输出中的表的键被缓冲，匹配的行从出现`Using join buffer`的行所代表的表中批量获取。

    在 JSON 格式输出中，`using_join_buffer`的值始终是`Block Nested Loop`、`Batched Key Access`或`hash join`之一。

    MySQL 8.0.18 开始提供哈希连接；MySQL 8.0.20 或更高版本的 MySQL 不使用块嵌套循环算法。有关这些优化的更多信息，请参见第 10.2.1.4 节，“哈希连接优化”和块嵌套循环连接算法。

    有关批量键访问算法的信息，请参见批量键访问连接。

+   `Using MRR`（JSON 属性：`message`）

    表使用多范围读取优化策略进行读取。请参见第 10.2.1.11 节，“多范围读取优化”。

+   `Using sort_union(...)`，`Using union(...)`，`Using intersect(...)`（JSON 属性：`message`）

    这些指示了特定算法，显示了如何为`index_merge`连接类型合并索引扫描。请参见第 10.2.1.3 节，“索引合并优化”。

+   `Using temporary`（JSON 属性：`using_temporary_table`）

    要解决查询，MySQL 需要创建一个临时表来保存结果。如果查询包含不同列的`GROUP BY`和`ORDER BY`子句，通常会发生这种情况。

+   `Using where`（JSON 属性：`attached_condition`）

    `WHERE`子句用于限制要与下一个表匹配或发送到客户端的行。除非您明确打算从表中获取或检查所有行，否则如果`Extra`值不是`Using where`且表连接类型为`ALL`或`index`，则查询可能存在问题。

    `Using where`在 JSON 格式输出中没有直接对应项；`attached_condition`属性包含使用的任何`WHERE`条件。

+   `Using where with pushed condition`（JSON 属性：`message`）

    此项仅适用于`NDB`表。这意味着 NDB Cluster 正在使用条件下推优化来提高非索引列和常量之间直接比较的效率。在这种情况下，条件被“推送”到集群的数据节点，并同时在所有数据节点上进行评估。这消除了在网络上传送不匹配的行的需要，并且可以将这类查询的速度提高 5 到 10 倍，相对于可能但未使用条件下推的情况。有关更多信息，请参见 Section 10.2.1.5, “Engine Condition Pushdown Optimization”。

+   `Zero limit`（JSON 属性：`message`）

    查询有一个`LIMIT 0`子句，无法选择任何行。

#### 解释输出解释

通过将`EXPLAIN`输出中`rows`列中的值相乘，您可以很好地了解连接的好坏。这应该大致告诉您 MySQL 必须检查多少行才能执行查询。如果使用`max_join_size`系统变量限制查询，这个行乘积也用于确定要执行哪些多表`SELECT`语句以及要中止哪些。请参见 Section 7.1.1, “Configuring the Server”。

以下示例展示了如何根据`EXPLAIN`提供的信息逐步优化多表连接。

假设你有以下所示的`SELECT`语句，并计划使用`EXPLAIN`来检查它：

```sql
EXPLAIN SELECT tt.TicketNumber, tt.TimeIn,
               tt.ProjectReference, tt.EstimatedShipDate,
               tt.ActualShipDate, tt.ClientID,
               tt.ServiceCodes, tt.RepetitiveID,
               tt.CurrentProcess, tt.CurrentDPPerson,
               tt.RecordVolume, tt.DPPrinted, et.COUNTRY,
               et_1.COUNTRY, do.CUSTNAME
        FROM tt, et, et AS et_1, do
        WHERE tt.SubmitTime IS NULL
          AND tt.ActualPC = et.EMPLOYID
          AND tt.AssignedPC = et_1.EMPLOYID
          AND tt.ClientID = do.CUSTNMBR;
```

对于此示例，请做出以下假设：

+   要比较的列已声明如下。

    | 表 | 列 | 数据类型 |
    | --- | --- | --- |
    | `tt` | `ActualPC` | `CHAR(10)` |
    | `tt` | `AssignedPC` | `CHAR(10)` |
    | `tt` | `ClientID` | `CHAR(10)` |
    | `et` | `EMPLOYID` | `CHAR(15)` |
    | `do` | `CUSTNMBR` | `CHAR(15)` |

+   表具有以下索引。

    | 表 | 索引 |
    | --- | --- |
    | `tt` | `ActualPC` |
    | `tt` | `AssignedPC` |
    | `tt` | `ClientID` |
    | `et` | `EMPLOYID`（主键） |
    | `do` | `CUSTNMBR`（主键） |

+   `tt.ActualPC`的值分布不均匀。

最初，在执行任何优化之前，`EXPLAIN`语句产生以下信息：

```sql
table type possible_keys key  key_len ref  rows  Extra
et    ALL  PRIMARY       NULL NULL    NULL 74
do    ALL  PRIMARY       NULL NULL    NULL 2135
et_1  ALL  PRIMARY       NULL NULL    NULL 74
tt    ALL  AssignedPC,   NULL NULL    NULL 3872
           ClientID,
           ActualPC
      Range checked for each record (index map: 0x23)
```

因为每个表的 `type` 都是 `ALL`，这个输出表明 MySQL 正在生成所有表的笛卡尔积；也就是说，每一行的组合都要被检查。这需要很长时间，因为必须检查每个表中行数的乘积。对于这个案例，这个乘积是 74 × 2135 × 74 × 3872 = 45,268,558,720 行。如果表更大，你可以想象需要多长时间。

这里的一个问题是，如果声明为相同类型和大小，MySQL 可以更有效地使用列上的索引。在这种情况下，如果声明为相同大小，`VARCHAR` 和 `CHAR` 被视为相同。`tt.ActualPC` 声明为 `CHAR(10)`，`et.EMPLOYID` 声明为 `CHAR(15)`，因此存在长度不匹配。

要解决列长度不匹配的差异，使用 `ALTER TABLE` 将 `ActualPC` 从 10 个字符扩展到 15 个字符：

```sql
mysql> ALTER TABLE tt MODIFY ActualPC VARCHAR(15);
```

现在 `tt.ActualPC` 和 `et.EMPLOYID` 都是 `VARCHAR(15)`。再次执行 `EXPLAIN` 语句会产生这个结果：

```sql
table type   possible_keys key     key_len ref         rows    Extra
tt    ALL    AssignedPC,   NULL    NULL    NULL        3872    Using
             ClientID,                                         where
             ActualPC
do    ALL    PRIMARY       NULL    NULL    NULL        2135
      Range checked for each record (index map: 0x1)
et_1  ALL    PRIMARY       NULL    NULL    NULL        74
      Range checked for each record (index map: 0x1)
et    eq_ref PRIMARY       PRIMARY 15      tt.ActualPC 1
```

这不是完美的，但要好得多：`rows` 值的乘积减少了 74 倍。这个版本在几秒钟内执行。

可以进行第二次修改以消除 `tt.AssignedPC = et_1.EMPLOYID` 和 `tt.ClientID = do.CUSTNMBR` 比较的列长度不匹配：

```sql
mysql> ALTER TABLE tt MODIFY AssignedPC VARCHAR(15),
                      MODIFY ClientID   VARCHAR(15);
```

经过这种修改，`EXPLAIN` 产生了如下所示的输出：

```sql
table type   possible_keys key      key_len ref           rows Extra
et    ALL    PRIMARY       NULL     NULL    NULL          74
tt    ref    AssignedPC,   ActualPC 15      et.EMPLOYID   52   Using
             ClientID,                                         where
             ActualPC
et_1  eq_ref PRIMARY       PRIMARY  15      tt.AssignedPC 1
do    eq_ref PRIMARY       PRIMARY  15      tt.ClientID   1
```

到这一步，查询已经被优化得几乎尽可能好了。剩下的问题是，默认情况下，MySQL 假设 `tt.ActualPC` 列中的值是均匀分布的，而对于 `tt` 表来说并非如此。幸运的是，可以轻松地告诉 MySQL 分析键的分布情况：

```sql
mysql> ANALYZE TABLE tt;
```

在额外的索引信息下，连接是完美的，`EXPLAIN` 产生了这个结果：

```sql
table type   possible_keys key     key_len ref           rows Extra
tt    ALL    AssignedPC    NULL    NULL    NULL          3872 Using
             ClientID,                                        where
             ActualPC
et    eq_ref PRIMARY       PRIMARY 15      tt.ActualPC   1
et_1  eq_ref PRIMARY       PRIMARY 15      tt.AssignedPC 1
do    eq_ref PRIMARY       PRIMARY 15      tt.ClientID   1
```

`EXPLAIN` 输出中的 `rows` 列是 MySQL 连接优化器的一个估计。通过比较 `rows` 的乘积与查询返回的实际行数，检查这些数字是否接近真实情况。如果数字相差很大，你可能会通过在 `SELECT` 语句中使用 `STRAIGHT_JOIN` 并尝试以不同顺序列出表来获得更好的性能。（但是，`STRAIGHT_JOIN` 可能会阻止索引的使用，因为它禁用了半连接转换。参见 Section 10.2.2.1, “Optimizing IN and EXISTS Subquery Predicates with Semijoin Transformations”.）

在某些情况下，当使用子查询时，可以执行修改数据的语句；有关更多信息，请参见第 15.2.15.8 节，“派生表”时使用`EXPLAIN SELECT`。
