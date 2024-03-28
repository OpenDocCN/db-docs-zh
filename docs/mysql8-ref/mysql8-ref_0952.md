# 15.2.13 SELECT Statement

> 原文：[`dev.mysql.com/doc/refman/8.0/en/select.html`](https://dev.mysql.com/doc/refman/8.0/en/select.html)

15.2.13.1 SELECT ... INTO Statement

15.2.13.2 JOIN Clause

```sql
SELECT
    [ALL | DISTINCT | DISTINCTROW ]
    [HIGH_PRIORITY]
    [STRAIGHT_JOIN]
    [SQL_SMALL_RESULT] [SQL_BIG_RESULT] [SQL_BUFFER_RESULT]
    [SQL_NO_CACHE] [SQL_CALC_FOUND_ROWS]
    *select_expr* [, *select_expr*] ...
    [*into_option*]
    [FROM *table_references*
      [PARTITION *partition_list*]]
    [WHERE *where_condition*]
    [GROUP BY {*col_name* | *expr* | *position*}, ... [WITH ROLLUP]]
    [HAVING *where_condition*]
    [WINDOW *window_name* AS (*window_spec*)
        [, *window_name* AS (*window_spec*)] ...]
    [ORDER BY {*col_name* | *expr* | *position*}
      [ASC | DESC], ... [WITH ROLLUP]]
    [LIMIT {[*offset*,] *row_count* | *row_count* OFFSET *offset*}]
    [*into_option*]
    [FOR {UPDATE | SHARE}
        [OF *tbl_name* [, *tbl_name*] ...]
        [NOWAIT | SKIP LOCKED]
      | LOCK IN SHARE MODE]
    [*into_option*]

*into_option*: {
    INTO OUTFILE '*file_name*'
        [CHARACTER SET *charset_name*]
        *export_options*
  | INTO DUMPFILE '*file_name*'
  | INTO *var_name* [, *var_name*] ...
}
```

`SELECT` 用于检索从一个或多个表中选择的行，并且可以包括 `UNION` 操作和子查询。从 MySQL 8.0.31 开始，还支持 `INTERSECT` 和 `EXCEPT` 操作。`UNION`、`INTERSECT` 和 `EXCEPT` 运算符将在本节后面更详细地描述。另请参阅 Section 15.2.15, “Subqueries”。

`SELECT` 语句可以以 `WITH`") 子句开头，以定义在 `SELECT` 中可访问的常用表达式。请参阅 Section 15.2.20, “WITH (Common Table Expressions)”")。

`SELECT` 语句最常用的子句包括：

+   每个 *`select_expr`* 表示要检索的列。必须至少有一个 *`select_expr`*。

+   *`table_references`* 指示要检索行的表或表。其语法在 Section 15.2.13.2, “JOIN Clause” 中描述。

+   `SELECT` 支持使用 `PARTITION` 子句显式选择分区，后跟表名中的分区或子分区列表（或两者都有）在 *`table_reference`* 中（参见 Section 15.2.13.2, “JOIN Clause”）。在这种情况下，仅从列出的分区中选择行，忽略表的任何其他分区。有关更多信息和示例，请参阅 Section 26.5, “Partition Selection”。

+   如果给出 `WHERE` 子句，则指示行必须满足的条件或条件。*`where_condition`* 是一个表达式，对于要选择的每一行都会计算为 true。如果没有 `WHERE` 子句，则该语句选择所有行。

    在 `WHERE` 表达式中，您可以使用 MySQL 支持的任何函数和运算符，但不能使用聚合（组）函数。请参阅 Section 11.5, “Expressions”，以及 Chapter 14, *Functions and Operators*。

`SELECT` 也可用于检索计算而不参考任何表的行。

例如：

```sql
mysql> SELECT 1 + 1;
 -> 2
```

在没有引用任何表的情况下，您可以指定 `DUAL` 作为虚拟表名：

```sql
mysql> SELECT 1 + 1 FROM DUAL;
 -> 2
```

`DUAL`纯粹是为那些要求所有`SELECT`语句应具有`FROM`和可能其他子句的人方便而设计的。MySQL 可能会忽略这些子句。如果没有引用表，则 MySQL 不需要`FROM DUAL`。

通常，必须按照语法描述中显示的顺序给出使用的子句。例如，`HAVING`子句必须在任何`GROUP BY`子句之后和任何`ORDER BY`子句之前。如果存在`INTO`子句，则可以出现在语法描述指示的任何位置，但在给定语句中只能出现一次，而不是在多个位置。有关`INTO`的更多信息，请参见第 15.2.13.1 节，“SELECT ... INTO 语句”。

*`select_expr`*项列表包括指示要检索哪些列的选择列表。项指定列或表达式，或可以使用`*`-简写：

+   仅由单个未限定的`*`组成的选择列表可以用作从所有表中选择所有列的简写：

    ```sql
    SELECT * FROM t1 INNER JOIN t2 ...
    ```

+   `*`tbl_name`*.*` 可以用作从命名表中选择所有列的限定简写：

    ```sql
    SELECT t1.*, t2.* FROM t1 INNER JOIN t2 ...
    ```

+   如果表具有不可见列，则`*`和`*`tbl_name`*.*` 不包括它们。要包括不可见列，必须明确引用它们。

+   在选择列表中与其他项目一起使用未限定的`*`可能会产生解析错误。例如：

    ```sql
    SELECT id, * FROM t1
    ```

    为避免此问题，请使用限定的`*`tbl_name`*.*` 引用：

    ```sql
    SELECT id, t1.* FROM t1
    ```

    在选择列表中为每个表使用限定的`*`tbl_name`*.*` 引用：

    ```sql
    SELECT AVG(score), t1.* FROM t1 ...
    ```

以下列表提供了有关其他`SELECT`子句的其他信息：

+   可以使用`AS *`alias_name`*`为*`select_expr`*指定别名。别名用作表达式的列名，并且可以在`GROUP BY`、`ORDER BY`或`HAVING`子句中使用。例如：

    ```sql
    SELECT CONCAT(last_name,', ',first_name) AS full_name
      FROM mytable ORDER BY full_name;
    ```

    在使用标识符为*`select_expr`*指定别名时，`AS`关键字是可选的。前面的示例可以这样写：

    ```sql
    SELECT CONCAT(last_name,', ',first_name) full_name
      FROM mytable ORDER BY full_name;
    ```

    但是，由于`AS`是可选的，如果忘记在两个*`select_expr`*表达式之间加逗号，可能会出现一个微妙的问题：MySQL 将第二个解释为别名。例如，在以下语句中，`columnb`被视为别名：

    ```sql
    SELECT columna columnb FROM mytable;
    ```

    因此，养成在指定列别名时明确使用`AS`的习惯是一个好习惯。

    在`WHERE`子句中不允许引用列别名，因为在执行`WHERE`子句时可能尚未确定列值。请参见第 B.3.4.4 节，“列别名问题”。

+   `FROM *`table_references`*` 子句表示要检索行的表或表。如果命名多个表，则正在执行连接。有关连接语法的信息，请参见第 15.2.13.2 节，“JOIN 子句”。对于每个指定的表，您可以选择指定别名。

    ```sql
    *tbl_name* [[AS] *alias*] [*index_hint*]
    ```

    使用索引提示可以为优化器提供有关在查询处理期间如何选择索引的信息。有关指定这些提示的语法的描述，请参见第 10.9.4 节，“索引提示”。

    你可以使用`SET max_seeks_for_key=*`value`*`作为一种替代方法，强制 MySQL 优先选择键扫描而不是表扫描。参见第 7.1.8 节，“服务器系统变量”。

+   你可以将默认数据库中的表称为*`tbl_name`*，或者作为*`db_name`*.*`tbl_name`*来明确指定数据库。你可以将列称为*`col_name`*，*`tbl_name`*.*`col_name`*，或*`db_name`*.*`tbl_name`*.*`col_name`*。除非引用会产生歧义，否则不需要为列引用指定*`tbl_name`*或*`db_name`*.*`tbl_name`*前缀。参见第 11.2.2 节，“标识符限定符”，了解需要更明确的列引用形式的歧义示例。

+   可以使用`*`tbl_name`* AS *`alias_name`*`或*`tbl_name alias_name`*对表引用进行别名。以下语句是等效的：

    ```sql
    SELECT t1.name, t2.salary FROM employee AS t1, info AS t2
      WHERE t1.name = t2.name;

    SELECT t1.name, t2.salary FROM employee t1, info t2
      WHERE t1.name = t2.name;
    ```

+   选择输出的列可以在`ORDER BY`和`GROUP BY`子句中使用列名、列别名或列位置进行引用。列位置是整数，从 1 开始：

    ```sql
    SELECT college, region, seed FROM tournament
      ORDER BY region, seed;

    SELECT college, region AS r, seed AS s FROM tournament
      ORDER BY r, s;

    SELECT college, region, seed FROM tournament
      ORDER BY 2, 3;
    ```

    要按照倒序排序，将`DESC`（降序）关键字添加到`ORDER BY`子句中你要排序的列的名称中。默认是升序；可以使用`ASC`关键字明确指定。

    如果`ORDER BY`出现在带括号的查询表达式中，并且也应用在外部查询中，结果是未定义的，并且可能在 MySQL 的将来版本中发生变化。

    使用列位置已被弃用，因为该语法已从 SQL 标准中删除。

+   在 MySQL 8.0.13 之前，MySQL 支持了一个非标准语法扩展，允许为`GROUP BY`列使用显式的`ASC`或`DESC`标识符。MySQL 8.0.12 及更高版本支持带有分组函数的`ORDER BY`，因此不再需要使用此扩展。这也意味着在使用`GROUP BY`时可以对任意列进行排序，就像这样：

    ```sql
    SELECT a, b, COUNT(c) AS t FROM test_table GROUP BY a,b ORDER BY a,t DESC;
    ```

    截至 MySQL 8.0.13，不再支持`GROUP BY`扩展：不允许为`GROUP BY`列使用`ASC`或`DESC`标识符。

+   当你使用`ORDER BY`或`GROUP BY`对`SELECT`中的列进行排序时，服务器仅使用由`max_sort_length`系统变量指示的初始字节数对值进行排序。

+   MySQL 扩展了对`GROUP BY`的使用，允许选择未在`GROUP BY`子句中提及的字段。如果您的查询未获得预期结果，请阅读 Section 14.19，“Aggregate Functions”中关于`GROUP BY`的描述。

+   `GROUP BY`允许使用`WITH ROLLUP`修饰符。请参见 Section 14.19.2，“GROUP BY Modifiers”。

    以前，在具有`WITH ROLLUP`修饰符的查询中不允许使用`ORDER BY`。从 MySQL 8.0.12 开始取消了此限制。请参见 Section 14.19.2，“GROUP BY Modifiers”。

+   `HAVING`子句与`WHERE`子句一样，指定选择条件。`WHERE`子句指定选择列表中的列的条件，但不能引用聚合函数。`HAVING`子句指定对由`GROUP BY`子句形成的组的条件。查询结果仅包括满足`HAVING`条件的组。（如果没有`GROUP BY`存在，则所有行隐式形成单个聚合组。）

    `HAVING`子句几乎是在最后应用的，就在项目发送到客户端之前，没有优化。（`LIMIT`在`HAVING`之后应用。）

    SQL 标准要求`HAVING`只能引用`GROUP BY`子句中的列或聚合函数中使用的列。然而，MySQL 支持对此行为的扩展，并允许`HAVING`引用`SELECT`列表中的列以及外部子查询中的列。

    如果`HAVING`子句引用的列存在歧义，将发出警告。在以下语句中，`col2`存在歧义，因为它既用作别名又用作列名：

    ```sql
    SELECT COUNT(col1) AS col2 FROM t GROUP BY col2 HAVING col2 = 2;
    ```

    标准 SQL 行为优先，因此如果`HAVING`列名既在`GROUP BY`中使用，又作为选择列列表中的别名列，则优先使用`GROUP BY`列。

+   不要将应该在`WHERE`子句中的项目放在`HAVING`中。例如，不要写如下内容：

    ```sql
    SELECT *col_name* FROM *tbl_name* HAVING *col_name* > 0;
    ```

    改为写成：

    ```sql
    SELECT *col_name* FROM *tbl_name* WHERE *col_name* > 0;
    ```

+   `HAVING`子句可以引用聚合函数，而`WHERE`子句不能：

    ```sql
    SELECT user, MAX(salary) FROM users
      GROUP BY user HAVING MAX(salary) > 10;
    ```

    （在某些较旧版本的 MySQL 中不起作用。）

+   MySQL 允许重复列名。也就是说，可以有多个具有相同名称的*`select_expr`*。这是对标准 SQL 的扩展。因为 MySQL 还允许`GROUP BY`和`HAVING`引用*`select_expr`*值，这可能导致歧义：

    ```sql
    SELECT 12 AS a, a FROM t GROUP BY a;
    ```

    在该语句中，两列都具有名称`a`。为确保正确使用列进行分组，请为每个*`select_expr`*使用不同的名称。

+   如果存在`WINDOW`子句，则定义了可以被窗口函数引用的命名窗口。详情请参见 Section 14.20.4，“Named Windows”。

+   MySQL 通过在`ORDER BY`子句中搜索*`select_expr`*值，然后在`FROM`子句中的表列中搜索来解析未限定的列或别名引用。对于`GROUP BY`或`HAVING`子句，它会先在`FROM`子句中搜索，然后再在*`select_expr`*值中搜索。（对于`GROUP BY`和`HAVING`，这与 MySQL 5.0 之前的行为不同，该行为使用与`ORDER BY`相同的规则。）

+   `LIMIT`子句可用于限制`SELECT`语句返回的行数。`LIMIT`接受一个或两个数字参数，这两个参数必须都是非负整数常量，但有以下例外：

    +   在准备好的语句中，可以使用`?`占位符标记指定`LIMIT`参数。

    +   在存储程序中，可以使用整数值例程参数或本地变量指定`LIMIT`参数。

    使用两个参数时，第一个参数指定要返回的第一行的偏移量，第二个参数指定要返回的最大行数。初始行的偏移量为 0（而不是 1）：

    ```sql
    SELECT * FROM tbl LIMIT 5,10;  # Retrieve rows 6-15
    ```

    要检索从某个偏移量到结果集末尾的所有行，可以使用一个很大的数字作为第二个参数。以下语句检索从第 96 行到最后的所有行：

    ```sql
    SELECT * FROM tbl LIMIT 95,18446744073709551615;
    ```

    使用一个参数时，该值指定从结果集开头返回的行数：

    ```sql
    SELECT * FROM tbl LIMIT 5;     # Retrieve first 5 rows
    ```

    换句话说，`LIMIT *`row_count`*`等同于`LIMIT 0, *`row_count`*`。

    对于准备好的语句，可以使用占位符。以下语句从`tbl`表中返回一行：

    ```sql
    SET @a=1;
    PREPARE STMT FROM 'SELECT * FROM tbl LIMIT ?';
    EXECUTE STMT USING @a;
    ```

    以下语句从`tbl`表中返回第二到第六行：

    ```sql
    SET @skip=1; SET @numrows=5;
    PREPARE STMT FROM 'SELECT * FROM tbl LIMIT ?, ?';
    EXECUTE STMT USING @skip, @numrows;
    ```

    为了与 PostgreSQL 兼容，MySQL 还支持`LIMIT *`row_count`* OFFSET *`offset`*`语法。

    如果`LIMIT`出现在括号查询表达式中，并且也应用于外部查询，则结果是未定义的，并且可能在 MySQL 的将来版本中更改。

+   `SELECT ... INTO`形式的`SELECT`允许将查询结果写入文件或存储在变量中。更多信息，请参见 Section 15.2.13.1, “SELECT ... INTO Statement”。

+   如果在使用页面或行锁的存储引擎中使用`FOR UPDATE`，则查询检查的行将被写锁定，直到当前事务结束。

    你不能在`CREATE TABLE *`new_table`* SELECT ... FROM *`old_table`* ...`等语句中将`FOR UPDATE`作为`SELECT`的一部分。（如果尝试这样做，将会收到错误消息“在创建'*`new_table`*'时无法更新表'*`old_table`*'。”）

    `FOR SHARE` 和 `LOCK IN SHARE MODE` 设置共享锁，允许其他事务读取检查的行，但不允许更新或删除它们。 `FOR SHARE` 和 `LOCK IN SHARE MODE` 是等效的。但是，`FOR SHARE`，像 `FOR UPDATE` 一样，支持 `NOWAIT`，`SKIP LOCKED` 和 `OF *`tbl_name`*` 选项。 `FOR SHARE` 是 `LOCK IN SHARE MODE` 的替代，但 `LOCK IN SHARE MODE` 仍可用于向后兼容。

    `NOWAIT` 会导致 `FOR UPDATE` 或 `FOR SHARE` 查询立即执行，如果由于另一个事务持有的锁而无法获得行锁，则返回错误。

    `SKIP LOCKED` 会导致 `FOR UPDATE` 或 `FOR SHARE` 查询立即执行，从结果集中排除被另一个事务锁定的行。

    `NOWAIT` 和 `SKIP LOCKED` 选项对基于语句的复制不安全。

    注意

    跳过被锁定行的查询会返回数据的不一致视图。因此，`SKIP LOCKED` 不适用于一般的事务工作。但是，当多个会话访问相同的类似队列的表时，可以使用它来避免锁争用。

    `OF *`tbl_name`*` 适用于对指定表执行 `FOR UPDATE` 和 `FOR SHARE` 查询。例如：

    ```sql
    SELECT * FROM t1, t2 FOR SHARE OF t1 FOR UPDATE OF t2;
    ```

    当省略 `OF *`tbl_name`*` 时，查询块引用的所有表都会被锁定。因此，在不与另一个锁定子句结合使用 `OF *`tbl_name`*` 的情况下使用锁定子句会返回错误。在多个锁定子句中指定相同的表会返回错误。如果在 `SELECT` 语句中指定了别名作为表名，则锁定子句只能使用该别名。如果 `SELECT` 语句没有明确指定别名，则锁定子句只能指定实际表名。

    有关 `FOR UPDATE` 和 `FOR SHARE` 的更多信息，请参见 Section 17.7.2.4, “Locking Reads”。有关 `NOWAIT` 和 `SKIP LOCKED` 选项的更多信息，请参见 Locking Read Concurrency with NOWAIT and SKIP LOCKED。

在 `SELECT` 关键字之后，您可以使用许多修饰符来影响语句的操作。 `HIGH_PRIORITY`，`STRAIGHT_JOIN` 和以 `SQL_` 开头的修饰符是 MySQL 对标准 SQL 的扩展。

+   `ALL` 和 `DISTINCT` 修饰符指定是否应返回重复行。 `ALL`（默认）指定应返回所有匹配行，包括重复行。 `DISTINCT` 指定从结果集中删除重复行。指定两个修饰符是错误的。 `DISTINCTROW` 是 `DISTINCT` 的同义词。

    在 MySQL 8.0.12 及更高版本中，`DISTINCT` 可以与使用 `WITH ROLLUP` 的查询一起使用。 (Bug #87450, Bug #26640100)

+   `HIGH_PRIORITY` 给予`SELECT`比更新表的语句更高的优先级。你应该只对非常快速且必须一次完成的查询使用这个选项。在表被锁定以供读取时发出的`SELECT HIGH_PRIORITY`查询即使有一个更新语句在等待表空闲也会运行。这只影响只使用表级锁定的存储引擎（如`MyISAM`、`MEMORY`和`MERGE`）。

    `HIGH_PRIORITY` 不能与 `SELECT` 语句一起使用，这些语句是 `UNION` 的一部分。

+   `STRAIGHT_JOIN` 强制优化器按照 `FROM` 子句中列出的顺序连接表。如果优化器以非最佳顺序连接表，可以使用这个选项加快查询速度。`STRAIGHT_JOIN` 也可以在 *`table_references`* 列表中使用。参见第 15.2.13.2 节，“JOIN 子句”。

    `STRAIGHT_JOIN` 不适用于优化器将其视为`const`或`system`表的任何表。这样的表产生一行，是在查询执行的优化阶段读取的，并且在查询执行继续之前，其列的引用被替换为适当的列值。这些表在`EXPLAIN`显示的查询计划中首先出现。参见第 10.8.1 节，“使用 EXPLAIN 优化查询”。这个例外可能不适用于在外连接的`NULL`补充侧使用的`const`或`system`表（即`LEFT JOIN`的右侧表或`RIGHT JOIN`的左侧表）。

+   `SQL_BIG_RESULT` 或 `SQL_SMALL_RESULT` 可以与 `GROUP BY` 或 `DISTINCT` 一起使用，告诉优化器结果集有很多行或很小，分别。对于 `SQL_BIG_RESULT`，如果创建了磁盘临时表，MySQL 直接使用它们，并倾向于对 `GROUP BY` 元素使用排序而不是使用带有键的临时表。对于 `SQL_SMALL_RESULT`，MySQL 使用内存临时表来存储结果表，而不是使用排序。这通常不需要。

+   `SQL_BUFFER_RESULT` 强制结果放入临时表中。这有助于 MySQL 提前释放表锁，并在向客户端发送结果集需要很长时间的情况下提供帮助。这个修饰符只能用于顶层`SELECT`语句，不能用于子查询或后续的`UNION`。

+   `SQL_CALC_FOUND_ROWS` 告诉 MySQL 计算结果集中会有多少行，忽略任何 `LIMIT` 子句。然后可以使用 `SELECT FOUND_ROWS()` 检索行数。参见第 14.15 节，“信息函数”。

    注意

    `SQL_CALC_FOUND_ROWS` 查询修饰符和配套的`FOUND_ROWS()`函数在 MySQL 8.0.17 中已被弃用；预计它们将在未来的 MySQL 版本中被移除。请查看`FOUND_ROWS()`的描述以获取有关替代策略的信息。

+   在 MySQL 8.0 之前，`SQL_CACHE` 和 `SQL_NO_CACHE` 修饰符与查询缓存一起使用。查询缓存在 MySQL 8.0 中被移除。`SQL_CACHE` 修饰符也被移除。`SQL_NO_CACHE` 已被弃用，并且没有效果；预计它将在未来的 MySQL 版本中被移除。
