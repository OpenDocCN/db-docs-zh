> 原文：[`dev.mysql.com/doc/refman/8.0/en/invisible-columns.html`](https://dev.mysql.com/doc/refman/8.0/en/invisible-columns.html)

#### 15.1.20.10 不可见列

MySQL 从 MySQL 8.0.23 开始支持不可见列。不可见列通常对查询隐藏，但如果显式引用，则可以访问。在 MySQL 8.0.23 之前，所有列都是可见的。

作为不可见列可能有用的示例，假设一个应用程序使用`SELECT *`查询来访问表，并且必须继续工作而无需修改，即使表被修改以添加一个应用程序不希望存在的新列。在`SELECT *`查询中，`*`会计算所有表列，除了那些不可见的列，因此解决方案是将新列添加为不可见列。该列仍然对`SELECT *`查询“隐藏”，应用程序继续像以前一样工作。如果必要，新版本的应用程序可以通过显式引用来引用不可见列。

以下各节详细介绍了 MySQL 如何处理不可见列。

+   DDL 语句和不可见列

+   DML 语句和不可见列

+   不可见列元数据

+   二进制日志和不可见列

##### DDL 语句和不可见列

列默认为可见。要为新列明确指定可见性，请在`CREATE TABLE`或`ALTER TABLE`的列定义中使用`VISIBLE`或`INVISIBLE`关键字：

```sql
CREATE TABLE t1 (
  i INT,
  j DATE INVISIBLE
) ENGINE = InnoDB;
ALTER TABLE t1 ADD COLUMN k INT INVISIBLE;
```

要更改现有列的可见性，请在`ALTER TABLE`列修改子句中使用`VISIBLE`或`INVISIBLE`关键字之一：

```sql
ALTER TABLE t1 CHANGE COLUMN j j DATE VISIBLE;
ALTER TABLE t1 MODIFY COLUMN j DATE INVISIBLE;
ALTER TABLE t1 ALTER COLUMN j SET VISIBLE;
```

表必须至少有一个可见列。尝试使所有列不可见会产生错误。

不可见列支持通常的列属性：`NULL`、`NOT NULL`、`AUTO_INCREMENT`等。

生成列可以是不可见的。

索引定义可以命名不可见列，包括`PRIMARY KEY`和`UNIQUE`索引的定义。虽然表必须至少有一个可见列，但索引定义不需要有任何可见列。

从表中删除的不可见列会像通常一样从命名该列的任何索引定义中删除。

外键约束可以定义在不可见列上，并且外键约束可以引用不可见列。

`CHECK`约束可以定义在不可见列上。对于新的或修改的行，违反不可见列上的`CHECK`约束会产生错误。

`CREATE TABLE ... LIKE` 包含不可见列，并且在新表中也是不可见的。

`CREATE TABLE ... SELECT` 不包括不可见列，除非它们在`SELECT`部分中被明确引用。然而，即使被明确引用，现有表中不可见的列在新表中也是可见的：

```sql
mysql> CREATE TABLE t1 (col1 INT, col2 INT INVISIBLE);
mysql> CREATE TABLE t2 AS SELECT col1, col2 FROM t1;
mysql> SHOW CREATE TABLE t2\G
*************************** 1\. row ***************************
       Table: t2
Create Table: CREATE TABLE `t2` (
  `col1` int DEFAULT NULL,
  `col2` int DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```

如果要保留不可见性，请在`CREATE TABLE`部分的`CREATE TABLE ... SELECT`语句中为不可见列提供定义：

```sql
mysql> CREATE TABLE t1 (col1 INT, col2 INT INVISIBLE);
mysql> CREATE TABLE t2 (col2 INT INVISIBLE) AS SELECT col1, col2 FROM t1;
mysql> SHOW CREATE TABLE t2\G
*************************** 1\. row ***************************
       Table: t2
Create Table: CREATE TABLE `t2` (
  `col1` int DEFAULT NULL,
  `col2` int DEFAULT NULL /*!80023 INVISIBLE */
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```

视图可以通过在定义视图的`SELECT`语句中明确引用它们来引用不可见列。在定义引用该列的视图之后更改列的可见性不会改变视图行为。

##### DML 语句和不可见列

对于`SELECT`语句，除非在选择列表中明确引用，否则不可见列不会成为结果集的一部分。在选择列表中，`*`和`*`tbl_name`*.*`的简写不包括不可见列。自然连接不包括不可见列。

考虑以下语句序列：

```sql
mysql> CREATE TABLE t1 (col1 INT, col2 INT INVISIBLE);
mysql> INSERT INTO t1 (col1, col2) VALUES(1, 2), (3, 4);

mysql> SELECT * FROM t1;
+------+
| col1 |
+------+
|    1 |
|    3 |
+------+

mysql> SELECT col1, col2 FROM t1;
+------+------+
| col1 | col2 |
+------+------+
|    1 |    2 |
|    3 |    4 |
+------+------+
```

第一个`SELECT`在选择列表中不引用不可见列`col2`（因为`*`不包括不可见列），所以`col2`不会出现在语句结果中。第二个`SELECT`明确引用`col2`，因此该列会出现在结果中。

语句`TABLE t1` 产生与第一个`SELECT`语句相同的输出。由于在`TABLE`语句中无法指定列，因此`TABLE`永远不会显示不可见列。

对于创建新行的语句，除非显式引用并赋值，否则不可见列将被分配其隐式默认值。有关隐式默认值的信息，请参阅隐式默认值处理。

对于`INSERT`（和`REPLACE`，对于未替换的行），当缺少列列表、空列列表或不包括不可见列的非空列列表时，隐式默认赋值会发生：

```sql
CREATE TABLE t1 (col1 INT, col2 INT INVISIBLE);
INSERT INTO t1 VALUES(...);
INSERT INTO t1 () VALUES(...);
INSERT INTO t1 (col1) VALUES(...);
```

对于前两个`INSERT`语句，`VALUES()`列表必须为每个可见列提供一个值，而不包括不可见列。对于第三个`INSERT`语句，`VALUES()`列表必须提供与命名列数相同的值；当您使用`VALUES ROW()`而不是`VALUES()`时也是如此。

对于`LOAD DATA`和`LOAD XML`，如果缺少列列表或非空列列表不包括不可见列，则会发生隐式默认赋值。输入行不应包含不可见列的值。

对于前述语句，如果要为不可见列分配除隐式默认值之外的值，请在列列表中明确命名不可见列并为其提供值。

`INSERT INTO ... SELECT *`和`REPLACE INTO ... SELECT *`不包括不可见列，因为`*`不包括不可见列。隐式默认赋值如前所述发生。

对于根据`PRIMARY KEY`或`UNIQUE`索引中的值插入或忽略新行，或替换或修改现有行的语句，MySQL 将不可见列视为可见列的相同方式处理：不可见列参与键值比较。具体来说，如果新行与唯一键值的现有行具有相同的值，则无论索引列是可见还是不可见，这些行为都会发生：

+   使用`IGNORE`修饰符，`INSERT`、`LOAD DATA`和`LOAD XML`会忽略新行。

+   `REPLACE`用新行替换现有行。使用`REPLACE`修饰符，`LOAD DATA`和`LOAD XML`也是如此。

+   `INSERT ... ON DUPLICATE KEY UPDATE`更新现有行。

对于`UPDATE`语句更新不可见列，与可见列一样，需要命名并分配值。

##### 不可见列元数据

列是否可见的信息可以从信息模式`COLUMNS`表的`EXTRA`列或`SHOW COLUMNS`输出中获取。例如：

```sql
mysql> SELECT TABLE_NAME, COLUMN_NAME, EXTRA
       FROM INFORMATION_SCHEMA.COLUMNS
       WHERE TABLE_SCHEMA = 'test' AND TABLE_NAME = 't1';
+------------+-------------+-----------+
| TABLE_NAME | COLUMN_NAME | EXTRA     |
+------------+-------------+-----------+
| t1         | i           |           |
| t1         | j           |           |
| t1         | k           | INVISIBLE |
+------------+-------------+-----------+
```

列默认可见，因此在这种情况下，`EXTRA`不显示可见性信息。对于不可见列，`EXTRA`显示`INVISIBLE`。

`SHOW CREATE TABLE`在表定义中显示不可见列，版本特定注释中包含`INVISIBLE`关键字：

```sql
mysql> SHOW CREATE TABLE t1\G
*************************** 1\. row ***************************
       Table: t1
Create Table: CREATE TABLE `t1` (
  `i` int DEFAULT NULL,
  `j` int DEFAULT NULL,
  `k` int DEFAULT NULL /*!80023 INVISIBLE */
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```

**mysqldump**和`mysqlpump`使用`SHOW CREATE TABLE`，因此它们在转储表定义时包括不可见列。它们还在转储数据时包括不可见列值。

将转储文件重新加载到不支持不可见列的旧版本的 MySQL 中会忽略特定于版本的注释，从而将任何不可见列创建为可见列。

##### 二进制日志和不可见列

MySQL 在二进制日志中处理不可见列如下：

+   表创建事件为不可见列包括`INVISIBLE`属性。

+   不可见列在行事件中被视为可见列。根据`binlog_row_image`系统变量设置，如果需要，则包括它们。

+   应用行事件时，不可见列在行事件中被视为可见列。特别是，根据`slave_rows_search_algorithms`系统变量设置选择要使用的算法和索引。

+   不可见列在计算写入集时被视为可见列。特别是，写入集包括在不可见列上定义的索引。

+   **mysqlbinlog** 命令在列元数据中包含可见性。
