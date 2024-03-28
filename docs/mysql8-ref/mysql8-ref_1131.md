# 15.8.2 EXPLAIN Statement

> 原文：[`dev.mysql.com/doc/refman/8.0/en/explain.html`](https://dev.mysql.com/doc/refman/8.0/en/explain.html)

```sql
{EXPLAIN | DESCRIBE | DESC}
    *tbl_name* [*col_name* | *wild*]

{EXPLAIN | DESCRIBE | DESC}
    [*explain_type*]
    {*explainable_stmt* | FOR CONNECTION *connection_id*}

{EXPLAIN | DESCRIBE | DESC} ANALYZE [FORMAT = TREE] *select_statement*

*explain_type*: {
    FORMAT = *format_name*
}

*format_name*: {
    TRADITIONAL
  | JSON
  | TREE
}

*explainable_stmt*: {
    SELECT statement
  | TABLE statement
  | DELETE statement
  | INSERT statement
  | REPLACE statement
  | UPDATE statement
}
```

`DESCRIBE` 和 `EXPLAIN` 语句是同义词。在实践中，`DESCRIBE` 关键字更常用于获取有关表结构的信息，而 `EXPLAIN` 用于获取查询执行计划（即 MySQL 如何执行查询的解释）。

以下讨论使用 `DESCRIBE` 和 `EXPLAIN` 关键字，但 MySQL 解析器将它们视为完全同义词。

+   获取表结构信息

+   获取执行计划信息

+   使用 EXPLAIN ANALYZE 获取信息

#### 获取表结构信息

`DESCRIBE` 提供了关于表中列的信息：

```sql
mysql> DESCRIBE City;
+------------+----------+------+-----+---------+----------------+
| Field      | Type     | Null | Key | Default | Extra          |
+------------+----------+------+-----+---------+----------------+
| Id         | int(11)  | NO   | PRI | NULL    | auto_increment |
| Name       | char(35) | NO   |     |         |                |
| Country    | char(3)  | NO   | UNI |         |                |
| District   | char(20) | YES  | MUL |         |                |
| Population | int(11)  | NO   |     | 0       |                |
+------------+----------+------+-----+---------+----------------+
```

`DESCRIBE` 是 `SHOW COLUMNS` 的快捷方式。这些语句也显示视图的信息。`SHOW COLUMNS` 的描述提供了有关输出列的更多信息。请参阅 Section 15.7.7.5, “SHOW COLUMNS Statement”。

默认情况下，`DESCRIBE` 显示表中所有列的信息。*`col_name`*，如果提供，是表中列的名称。在这种情况下，语句仅显示指定列的信息。*`wild`*，如果提供，是一个模式字符串。它可以包含 SQL 的 `%` 和 `_` 通配符字符。在这种情况下，语句仅显示名称与字符串匹配的列的输出。除非字符串包含空格或其他特殊字符，否则无需将字符串括在引号内。

`DESCRIBE` 语句是为了与 Oracle 兼容而提供的。

`SHOW CREATE TABLE`、`SHOW TABLE STATUS` 和 `SHOW INDEX` 语句也提供有关表的信息。请参阅 Section 15.7.7, “SHOW Statements”。

`explain_format`系统变量在 MySQL 8.0.32 中添加，对于用于获取有关表列信息的`EXPLAIN`输出没有影响。

#### 获取执行计划信息

`EXPLAIN`语句提供有关 MySQL 如何执行语句的信息：

+   `EXPLAIN`适用于`SELECT`、`DELETE`、`INSERT`、`REPLACE`和`UPDATE`语句。在 MySQL 8.0.19 及更高版本中，它还适用于`TABLE`语句。

+   当使用`EXPLAIN`解释可解释的语句时，MySQL 会显示有关语句执行计划的优化器信息。也就是说，MySQL 会解释它将如何处理该语句，包括有关表如何连接以及连接顺序的信息。有关使用`EXPLAIN`获取执行计划信息的信息，请参见 Section 10.8.2, “EXPLAIN Output Format”。

+   当使用`EXPLAIN`与`FOR CONNECTION *`connection_id`*`而不是可解释的语句一起使用时，它会显示在指定连接中执行的语句的执行计划。请参见 Section 10.8.4, “Obtaining Execution Plan Information for a Named Connection”。

+   对于可解释的语句，`EXPLAIN`生成额外的执行计划信息，可以使用`SHOW WARNINGS`显示。请参见 Section 10.8.3, “Extended EXPLAIN Output Format”。

+   `EXPLAIN`对于检查涉及分区表的查询很有用��请参见 Section 26.3.5, “Obtaining Information About Partitions”。

+   `FORMAT`选项可用于选择输出格式。`TRADITIONAL`以表格形式呈现输出。如果没有`FORMAT`选项，则默认为此格式。`JSON`格式以 JSON 格式显示信息。在 MySQL 8.0.16 及更高版本中，`TREE`提供类似树状的输出，比`TRADITIONAL`格式更精确地描述了查询处理的方式；它是唯一显示哈希连接使用情况的格式（请参见 Section 10.2.1.4, “Hash Join Optimization”），并且始终用于`EXPLAIN ANALYZE`。

    截至 MySQL 8.0.32，`EXPLAIN`使用的默认输出格式（即，当没有`FORMAT`选项时）由`explain_format`系统变量的值确定。此变量的确切影响将在本节后面描述。

`EXPLAIN`需要执行解释语句所需的相同权限。此外，`EXPLAIN`还需要对任何解释的视图具有`SHOW VIEW`权限。如果指定的连接属于不同用户，则`EXPLAIN ... FOR CONNECTION`还需要`PROCESS`权限。

MySQL 8.0.32 中引入的`explain_format`系统变量确定在显示查询执行计划时`EXPLAIN`的输出格式。此变量可以采用与`FORMAT`选项一起使用的任何值，另外还添加了`DEFAULT`作为`TRADITIONAL`的同义词。以下示例使用`world`数据库中的`country`表，该表可以从 MySQL: Other Downloads 获取：

```sql
mysql> USE world; # Make world the current database
Database changed
```

检查`explain_format`的值，我们看到它具有默认值，因此`EXPLAIN`（没有`FORMAT`选项）因此使用传统的表格输出：

```sql
mysql> SELECT @@explain_format;
+------------------+
| @@explain_format |
+------------------+
| TRADITIONAL      |
+------------------+
1 row in set (0.00 sec)

mysql> EXPLAIN SELECT Name FROM country WHERE Code Like 'A%';
+----+-------------+---------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
| id | select_type | table   | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra       |
+----+-------------+---------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
|  1 | SIMPLE      | country | NULL       | range | PRIMARY       | PRIMARY | 12      | NULL |   17 |   100.00 | Using where |
+----+-------------+---------+------------+-------+---------------+---------+---------+------+------+----------+-------------+
1 row in set, 1 warning (0.00 sec)
```

如果将`explain_format`的值设置为`TREE`，然后重新运行相同的`EXPLAIN`语句，输出将使用类似树状的格式：

```sql
mysql> SET @@explain_format=TREE;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @@explain_format;
+------------------+
| @@explain_format |
+------------------+
| TREE             |
+------------------+
1 row in set (0.00 sec)

mysql> EXPLAIN SELECT Name FROM country WHERE Code LIKE 'A%';
+--------------------------------------------------------------------------------------------------------------+
| EXPLAIN                                                                                                      |
+--------------------------------------------------------------------------------------------------------------+
| -> Filter: (country.`Code` like 'A%')  (cost=3.67 rows=17)
 -> Index range scan on country using PRIMARY over ('A' <= Code <= 'A????????')  (cost=3.67 rows=17)  |
+--------------------------------------------------------------------------------------------------------------+
1 row in set, 1 warning (0.00 sec)
```

如前所述，`FORMAT`选项会覆盖此设置。使用`FORMAT=JSON`而不是`FORMAT=TREE`执行相同的`EXPLAIN`语句，可以看到这一点：

```sql
mysql> EXPLAIN FORMAT=JSON SELECT Name FROM country WHERE Code LIKE 'A%';
+------------------------------------------------------------------------------+
| EXPLAIN                                                                      |
+------------------------------------------------------------------------------+
| {
  "query_block": {
    "select_id": 1,
    "cost_info": {
      "query_cost": "3.67"
    },
    "table": {
      "table_name": "country",
      "access_type": "range",
      "possible_keys": [
        "PRIMARY"
      ],
      "key": "PRIMARY",
      "used_key_parts": [
        "Code"
      ],
      "key_length": "12",
      "rows_examined_per_scan": 17,
      "rows_produced_per_join": 17,
      "filtered": "100.00",
      "cost_info": {
        "read_cost": "1.97",
        "eval_cost": "1.70",
        "prefix_cost": "3.67",
        "data_read_per_join": "16K"
      },
      "used_columns": [
        "Code",
        "Name"
      ],
      "attached_condition": "(`world`.`country`.`Code` like 'A%')"
    }
  }
}                                                                              |
+------------------------------------------------------------------------------+
1 row in set, 1 warning (0.00 sec)
```

要将`EXPLAIN`的默认输出返回到表格格式，请将`explain_format`设置为`TRADITIONAL`。或者，您可以将其设置为`DEFAULT`，效果相同，如下所示：

```sql
mysql> SET @@explain_format=DEFAULT;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @@explain_format;
+------------------+
| @@explain_format |
+------------------+
| TRADITIONAL      |
+------------------+
1 row in set (0.00 sec)
```

借助`EXPLAIN`，您可以看到应该在哪些表上添加索引，以便通过使用索引查找行来使语句执行更快。您还可以使用`EXPLAIN`来检查优化器是否以最佳顺序连接表。为了提示优化器使用与在`SELECT`语句中命名表的顺序相对应的连接顺序，可以在语句开头使用`SELECT STRAIGHT_JOIN`而不仅仅是`SELECT`。（参见 Section 15.2.13, “SELECT Statement”.）

优化器跟踪有时可能提供与 `EXPLAIN` 不同的信息。但是，优化器跟踪的格式和内容可能会在版本之间发生变化。有关详细信息，请参见 MySQL Internals: Tracing the Optimizer。

如果您发现索引没有被使用，而您认为它们应该被使用，请运行 `ANALYZE TABLE` 来更新表统计信息，例如键的基数，这可能会影响优化器的选择。请参阅 Section 15.7.3.1, “ANALYZE TABLE Statement”。

注意

MySQL Workbench 具有可视化解释功能，提供 `EXPLAIN` 输出的可视化表示。请参阅 教程：使用 Explain 改进查询性能。

#### 使用 EXPLAIN ANALYZE 获取信息

MySQL 8.0.18 引入了 `EXPLAIN ANALYZE`，它运行一个语句并生成带有时间和额外基于迭代器的信息的 `EXPLAIN` 输出，展示优化器的期望与实际执行的匹配情况。对于每个迭代器，提供以下信息：

+   预估执行成本

    （某些迭代器不受成本模型考虑，因此不包括在估计中。）

+   预估返回的行数

+   返回第一行所需的时间

+   执行此迭代器所花费的时间（包括子迭代器，但不包括父迭代器），以毫秒表示。

    （当存在多个循环时，此数字显示每个循环的平均时间。）

+   迭代器返回的行数

+   循环次数

查询执行信息使用 `TREE` 输出格式显示，其中节点表示迭代器。`EXPLAIN ANALYZE` 总是使用 `TREE` 输出格式。在 MySQL 8.0.21 及更高版本中，可以选择使用 `FORMAT=TREE` 明确指定；不支持除 `TREE` 之外的其他格式。

`EXPLAIN ANALYZE` 可以与 `SELECT` 语句一起使用，也可以与多表 `UPDATE` 和 `DELETE` 语句一起使用。从 MySQL 8.0.19 开始，还可以与 `TABLE` 语句一起使用。

从 MySQL 8.0.20 开始，您可以使用 `KILL QUERY` 或 **CTRL-C** 终止此语句。

`EXPLAIN ANALYZE` 不能与 `FOR CONNECTION` 一起使用。

示例输出：

```sql
mysql> EXPLAIN ANALYZE SELECT * FROM t1 JOIN t2 ON (t1.c1 = t2.c2)\G
*************************** 1\. row ***************************
EXPLAIN: -> Inner hash join (t2.c2 = t1.c1)  (cost=4.70 rows=6)
(actual time=0.032..0.035 rows=6 loops=1)
 -> Table scan on t2  (cost=0.06 rows=6)
(actual time=0.003..0.005 rows=6 loops=1)
 -> Hash
 -> Table scan on t1  (cost=0.85 rows=6)
(actual time=0.018..0.022 rows=6 loops=1)

mysql> EXPLAIN ANALYZE SELECT * FROM t3 WHERE i > 8\G
*************************** 1\. row ***************************
EXPLAIN: -> Filter: (t3.i > 8)  (cost=1.75 rows=5)
(actual time=0.019..0.021 rows=6 loops=1)
 -> Table scan on t3  (cost=1.75 rows=15)
(actual time=0.017..0.019 rows=15 loops=1)

mysql> EXPLAIN ANALYZE SELECT * FROM t3 WHERE pk > 17\G
*************************** 1\. row ***************************
EXPLAIN: -> Filter: (t3.pk > 17)  (cost=1.26 rows=5)
(actual time=0.013..0.016 rows=5 loops=1)
 -> Index range scan on t3 using PRIMARY  (cost=1.26 rows=5)
(actual time=0.012..0.014 rows=5 loops=1)
```

示例输出中使用的表是通过以下显示的语句创建的：

```sql
CREATE TABLE t1 (
    c1 INTEGER DEFAULT NULL,
    c2 INTEGER DEFAULT NULL
);

CREATE TABLE t2 (
    c1 INTEGER DEFAULT NULL,
    c2 INTEGER DEFAULT NULL
);

CREATE TABLE t3 (
    pk INTEGER NOT NULL PRIMARY KEY,
    i INTEGER DEFAULT NULL
);
```

输出中显示的 `actual time` 的值以毫秒表示。

截至 MySQL 8.0.32，`explain_format`系统变量对`EXPLAIN ANALYZE`有以下影响：

+   如果此变量的值为`TRADITIONAL`或`TREE`（或同义词`DEFAULT`），`EXPLAIN ANALYZE`将使用`TREE`格式。这确保了该语句继续默认使用`TREE`格式，就像在引入`explain_format`之前一样。

+   如果`explain_format`的值为`JSON`，则除非在语句中指定`FORMAT=TREE`，否则`EXPLAIN ANALYZE`会返回错误。这是因为`EXPLAIN ANALYZE`仅支持`TREE`输出格式。

我们在这里说明了第二点描述的行为，重复使用了前一个示例中的最后一个`EXPLAIN ANALYZE`语句：

```sql
mysql> SET @@explain_format=JSON;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT @@explain_format;
+------------------+
| @@explain_format |
+------------------+
| JSON             |
+------------------+
1 row in set (0.00 sec)

mysql> EXPLAIN ANALYZE SELECT * FROM t3 WHERE pk > 17\G
ERROR 1235 (42000): This version of MySQL doesn't yet support 'EXPLAIN ANALYZE with JSON format' 
mysql> EXPLAIN ANALYZE FORMAT=TRADITIONAL SELECT * FROM t3 WHERE pk > 17\G
ERROR 1235 (42000): This version of MySQL doesn't yet support 'EXPLAIN ANALYZE with TRADITIONAL format' 
mysql> EXPLAIN ANALYZE FORMAT=TREE SELECT * FROM t3 WHERE pk > 17\G
*************************** 1\. row ***************************
EXPLAIN: -> Filter: (t3.pk > 17)  (cost=1.26 rows=5)
(actual time=0.013..0.016 rows=5 loops=1)
 -> Index range scan on t3 using PRIMARY  (cost=1.26 rows=5)
(actual time=0.012..0.014 rows=5 loops=1)
```

使用`FORMAT=TRADITIONAL`或`FORMAT=JSON`与`EXPLAIN ANALYZE`总是会引发错误，无论`explain_format`的值如何。

从 MySQL 8.0.33 开始，`EXPLAIN ANALYZE`和`EXPLAIN FORMAT=TREE`输出中的数字将根据以下规则进行格式化：

+   0.001-999999.5 范围内的数字以十进制数形式打印。

    小于 1000 的十进制数有三个有效数字；其余的有四、五或六个。

+   超出 0.001-999999.5 范围的数字以工程格式打印。这些值的示例是`1.23e+9`和`934e-6`。

+   不会打印尾随的零。例如，我们打印`2.3`而不是`2.30`，打印`1.2e+6`而不是`1.20e+6`。

+   小于`1e-12`的数字打印为`0`。
