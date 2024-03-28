> 原文：[`dev.mysql.com/doc/refman/8.0/en/partitioning-columns-range.html`](https://dev.mysql.com/doc/refman/8.0/en/partitioning-columns-range.html)

#### 26.2.3.1 RANGE COLUMNS 分区

列范围分区类似于范围分区，但允许您根据多个列值的范围定义分区。此外，您可以使用非整数类型的列定义范围。

`RANGE COLUMNS`分区与`RANGE`分区在以下方面有显著不同：

+   `RANGE COLUMNS`不接受表达式，只接受列名。

+   `RANGE COLUMNS`接受一个或多个列的列表。

    `RANGE COLUMNS`分区基于元组（列值列表）之间的比较，而不是标量值之间的比较。将行放置在`RANGE COLUMNS`分区中也是基于元组之间的比较；这将在本节后面进一步讨论。

+   `RANGE COLUMNS`分区列不限于整数列；字符串、`DATE`和`DATETIME`列也可以用作分区列。（详细信息请参阅第 26.2.3 节，“COLUMNS Partitioning”。）

创建由`RANGE COLUMNS`分区的基本语法如下所示：

```sql
CREATE TABLE *table_name*
PARTITION BY RANGE COLUMNS(*column_list*) (
    PARTITION *partition_name* VALUES LESS THAN (*value_list*)[,
    PARTITION *partition_name* VALUES LESS THAN (*value_list*)][,
    ...]
)

*column_list*:
    *column_name*[, *column_name*][, ...]

*value_list*:
    *value*[, *value*][, ...]
```

注意

在创建分区表时可以使用的并非所有`CREATE TABLE`选项都在此处展示。有关完整信息，请参阅第 15.1.20 节，“CREATE TABLE Statement”。

在刚刚展示的语法中，*`column_list`*是一个或多个列的列表（有时称为分区列列表），*`value_list`*是一个值列表（即，它是一个分区定义值列表）。必须为每个分区定义提供一个*`value_list`*，并且每个*`value_list`*必须具有与*`column_list`*中列数相同的值。一般来说，如果在`COLUMNS`子句中使用了*`N`*列，则每个`VALUES LESS THAN`子句也必须提供一个包含*`N`*个值的列表。

分区列列表中的元素和定义每个分区的值列表中的元素必须以相同的顺序出现。此外，值列表中的每个元素必须与列列表中的相应元素具有相同的数据类型。然而，分区列列表和值列表中列名的顺序不必与`CREATE TABLE`语句的主体部分中表列定义的顺序相同。与通过`RANGE`分区的表一样，您可以使用`MAXVALUE`来表示一个值，使得插入到给定列中的任何合法值始终小于此值。以下是一个`CREATE TABLE`语句的示例，可帮助说明所有这些要点：

```sql
mysql> CREATE TABLE rcx (
 ->     a INT,
 ->     b INT,
 ->     c CHAR(3),
 ->     d INT
 -> )
 -> PARTITION BY RANGE COLUMNS(a,d,c) (
 ->     PARTITION p0 VALUES LESS THAN (5,10,'ggg'),
 ->     PARTITION p1 VALUES LESS THAN (10,20,'mmm'),
 ->     PARTITION p2 VALUES LESS THAN (15,30,'sss'),
 ->     PARTITION p3 VALUES LESS THAN (MAXVALUE,MAXVALUE,MAXVALUE)
 -> );
Query OK, 0 rows affected (0.15 sec)
```

表`rcx`包含列`a`、`b`、`c`、`d`。提供给`COLUMNS`子句的分区列列表使用了这些列中的 3 列，顺序为`a`、`d`、`c`。用于定义分区的每个值列表包含 3 个值，顺序相同；也就是说，每个值列表元组的形式为（`INT`、`INT`、`CHAR(3)`），这对应于列`a`、`d`和`c`使用的数据类型（按顺序）。

将行放入分区是通过比较要插入的行中与`COLUMNS`子句中匹配的元组与用于定义表分区的`VALUES LESS THAN`子句中使用的元组来确定的。因为我们比较的是元组（即值的列表或集合），而不是标量值，所以在与简单的`RANGE`分区不同的情况下，与`RANGE COLUMNS`分区一起使用的`VALUES LESS THAN`的语义有所不同。在`RANGE`分区中，生成与`VALUES LESS THAN`中的限制值相等的表达式值的行永远不会放入相应的分区；然而，在使用`RANGE COLUMNS`分区时，有时可能会将分区列列表的第一个元素的值与`VALUES LESS THAN`值列表中第一个元素的值相等的行放入相应的分区。

考虑通过以下语句创建的`RANGE`分区表：

```sql
CREATE TABLE r1 (
    a INT,
    b INT
)
PARTITION BY RANGE (a)  (
    PARTITION p0 VALUES LESS THAN (5),
    PARTITION p1 VALUES LESS THAN (MAXVALUE)
);
```

如果我们向该表中插入 3 行，使得每行的`a`列值均为`5`，则所有 3 行都存储在分区`p1`中，因为在每种情况下，`a`列值均不小于 5，我们可以通过针对信息模式`PARTITIONS`表执行适当的查询来查看：

```sql
mysql> INSERT INTO r1 VALUES (5,10), (5,11), (5,12);
Query OK, 3 rows affected (0.00 sec)
Records: 3  Duplicates: 0  Warnings: 0

mysql> SELECT PARTITION_NAME, TABLE_ROWS
 ->     FROM INFORMATION_SCHEMA.PARTITIONS
 ->     WHERE TABLE_NAME = 'r1';
+----------------+------------+
| PARTITION_NAME | TABLE_ROWS |
+----------------+------------+
| p0             |          0 |
| p1             |          3 |
+----------------+------------+
2 rows in set (0.00 sec)
```

现在考虑一个类似的表`rc1`，它使用了`RANGE COLUMNS`分区，`COLUMNS`子句中引用了列`a`和`b`，如下所示创建：

```sql
CREATE TABLE rc1 (
    a INT,
    b INT
)
PARTITION BY RANGE COLUMNS(a, b) (
    PARTITION p0 VALUES LESS THAN (5, 12),
    PARTITION p3 VALUES LESS THAN (MAXVALUE, MAXVALUE)
);
```

如果我们将刚刚插入`r1`的相同行插入`rc1`，则行的分布会有所不同：

```sql
mysql> INSERT INTO rc1 VALUES (5,10), (5,11), (5,12);
Query OK, 3 rows affected (0.00 sec)
Records: 3  Duplicates: 0  Warnings: 0

mysql> SELECT PARTITION_NAME, TABLE_ROWS
 ->     FROM INFORMATION_SCHEMA.PARTITIONS
 ->     WHERE TABLE_NAME = 'rc1';
+----------------+------------+
| PARTITION_NAME | TABLE_ROWS |
+----------------+------------+
| p0             |          2 |
| p3             |          1 |
+----------------+------------+
2 rows in set (0.00 sec)
```

这是因为我们比较的是行而不是标量值。我们可以将插入的行值与用于在表`rc1`中定义分区`p0`的`VALUES THAN LESS THAN`子句中的限制行值进行比较，如下所示：

```sql
mysql> SELECT (5,10) < (5,12), (5,11) < (5,12), (5,12) < (5,12);
+-----------------+-----------------+-----------------+
| (5,10) < (5,12) | (5,11) < (5,12) | (5,12) < (5,12) |
+-----------------+-----------------+-----------------+
|               1 |               1 |               0 |
+-----------------+-----------------+-----------------+
1 row in set (0.00 sec)
```

2 元组`(5,10)`和`(5,11)`被认为小于`(5,12)`，因此它们被存储在分区`p0`中。由于 5 不小于 5，12 不小于 12，`(5,12)`被认为不小于`(5,12)`，并存储在分区`p1`中。

在前面的示例中，`SELECT`语句也可以使用显式行构造函数编写，如下所示：

```sql
SELECT ROW(5,10) < ROW(5,12), ROW(5,11) < ROW(5,12), ROW(5,12) < ROW(5,12);
```

有关在 MySQL 中使用行构造函数的更多信息，请参阅第 15.2.15.5 节，“行子查询”。

对于只使用单个分区列进行`RANGE COLUMNS`分区的表，行存储在分区中的方式与通过`RANGE`分区的等效表相同。以下`CREATE TABLE`语句创建了一个使用 1 个分区列进行`RANGE COLUMNS`分区的表：

```sql
CREATE TABLE rx (
    a INT,
    b INT
)
PARTITION BY RANGE COLUMNS (a)  (
    PARTITION p0 VALUES LESS THAN (5),
    PARTITION p1 VALUES LESS THAN (MAXVALUE)
);
```

如果我们将行`(5,10)`、`(5,11)`和`(5,12)`插入到这个表中，我们可以看到它们的放置方式与我们之前创建和填充的表`r`相同：

```sql
mysql> INSERT INTO rx VALUES (5,10), (5,11), (5,12);
Query OK, 3 rows affected (0.00 sec)
Records: 3  Duplicates: 0  Warnings: 0

mysql> SELECT PARTITION_NAME,TABLE_ROWS
 ->     FROM INFORMATION_SCHEMA.PARTITIONS
 ->     WHERE TABLE_NAME = 'rx';
+----------------+------------+
| PARTITION_NAME | TABLE_ROWS |
+----------------+------------+
| p0             |          0 |
| p1             |          3 |
+----------------+------------+
2 rows in set (0.00 sec)
```

也可以创建按`RANGE COLUMNS`分区的表，其中一个或多个列的限制值在连续的分区定义中重复。只要用于定义分区的列值元组严格递增，就可以这样做。例如，以下每个`CREATE TABLE`语句都是有效的：

```sql
CREATE TABLE rc2 (
    a INT,
    b INT
)
PARTITION BY RANGE COLUMNS(a,b) (
    PARTITION p0 VALUES LESS THAN (0,10),
    PARTITION p1 VALUES LESS THAN (10,20),
    PARTITION p2 VALUES LESS THAN (10,30),
    PARTITION p3 VALUES LESS THAN (MAXVALUE,MAXVALUE)
 );

CREATE TABLE rc3 (
    a INT,
    b INT
)
PARTITION BY RANGE COLUMNS(a,b) (
    PARTITION p0 VALUES LESS THAN (0,10),
    PARTITION p1 VALUES LESS THAN (10,20),
    PARTITION p2 VALUES LESS THAN (10,30),
    PARTITION p3 VALUES LESS THAN (10,35),
    PARTITION p4 VALUES LESS THAN (20,40),
    PARTITION p5 VALUES LESS THAN (MAXVALUE,MAXVALUE)
 );
```

即使乍看之下可能不会成功，以下语句也会成功，因为列`b`的限制值对于分区`p0`为 25，对于分区`p1`为 20，列`c`的限制值对于分区`p1`为 100，对于分区`p2`为 50：

```sql
CREATE TABLE rc4 (
    a INT,
    b INT,
    c INT
)
PARTITION BY RANGE COLUMNS(a,b,c) (
    PARTITION p0 VALUES LESS THAN (0,25,50),
    PARTITION p1 VALUES LESS THAN (10,20,100),
    PARTITION p2 VALUES LESS THAN (10,30,50),
    PARTITION p3 VALUES LESS THAN (MAXVALUE,MAXVALUE,MAXVALUE)
 );
```

在设计按`RANGE COLUMNS`分区的表时，您可以通过使用**mysql**客户端对所需元组进行比较来测试连续的分区定义，如下所示：

```sql
mysql> SELECT (0,25,50) < (10,20,100), (10,20,100) < (10,30,50);
+-------------------------+--------------------------+
| (0,25,50) < (10,20,100) | (10,20,100) < (10,30,50) |
+-------------------------+--------------------------+
|                       1 |                        1 |
+-------------------------+--------------------------+
1 row in set (0.00 sec)
```

如果`CREATE TABLE`语句包含不严格递增顺序的分区定义，它将失败并显示错误，如下例所示：

```sql
mysql> CREATE TABLE rcf (
 ->     a INT,
 ->     b INT,
 ->     c INT
 -> )
 -> PARTITION BY RANGE COLUMNS(a,b,c) (
 ->     PARTITION p0 VALUES LESS THAN (0,25,50),
 ->     PARTITION p1 VALUES LESS THAN (20,20,100),
 ->     PARTITION p2 VALUES LESS THAN (10,30,50),
 ->     PARTITION p3 VALUES LESS THAN (MAXVALUE,MAXVALUE,MAXVALUE)
 ->  );
ERROR 1493 (HY000): VALUES LESS THAN value must be strictly increasing for each partition
```

当你遇到这样的错误时，可以通过对它们的列列表进行“小于”比较来推断哪些分区定义是无效的。在这种情况下，问题出在分区`p2`的定义上，因为用于定义它的元组不小于用于定义分区`p3`的元组，如下所示：

```sql
mysql> SELECT (0,25,50) < (20,20,100), (20,20,100) < (10,30,50);
+-------------------------+--------------------------+
| (0,25,50) < (20,20,100) | (20,20,100) < (10,30,50) |
+-------------------------+--------------------------+
|                       1 |                        0 |
+-------------------------+--------------------------+
1 row in set (0.00 sec)
```

当使用`RANGE COLUMNS`时，同一列中的`MAXVALUE`可能出现在多个`VALUES LESS THAN`子句中。但是，连续分区定义中各列的限制值应该是递增的，不应该定义超过一个分区，其中`MAXVALUE`用作所有列值的上限，并且此分区定义应该出现在`PARTITION ... VALUES LESS THAN`子句列表的最后。此外，您不能将`MAXVALUE`用作连续分区定义中第一列的限制值。

如前所述，使用`RANGE COLUMNS`分区还可以使用非整数列作为分区列。（有关这些列的完整列表，请参阅第 26.2.3 节，“列分区”。）考虑一个名为`employees`的表（未分区），使用以下语句创建：

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT NOT NULL,
    store_id INT NOT NULL
);
```

使用`RANGE COLUMNS`分区，您可以创建这个表的一个版本，根据员工的姓氏将每一行存储在四个分区中的一个，就像这样：

```sql
CREATE TABLE employees_by_lname (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT NOT NULL,
    store_id INT NOT NULL
)
PARTITION BY RANGE COLUMNS (lname)  (
    PARTITION p0 VALUES LESS THAN ('g'),
    PARTITION p1 VALUES LESS THAN ('m'),
    PARTITION p2 VALUES LESS THAN ('t'),
    PARTITION p3 VALUES LESS THAN (MAXVALUE)
);
```

或者，您可以使用以下`ALTER TABLE`语句使之前创建的`employees`表按照这种方案进行分区。

```sql
ALTER TABLE employees PARTITION BY RANGE COLUMNS (lname)  (
    PARTITION p0 VALUES LESS THAN ('g'),
    PARTITION p1 VALUES LESS THAN ('m'),
    PARTITION p2 VALUES LESS THAN ('t'),
    PARTITION p3 VALUES LESS THAN (MAXVALUE)
);
```

注意

因为不同的字符集和校对规则具有不同的排序顺序，所以在使用字符串列作为分区列进行分区时，正在使用的字符集和校对规则可能会影响表按`RANGE COLUMNS`分区的哪个分区存储给定行。此外，在创建这样一个表之后更改给定数据库、表或列的字符集或校对规则可能会导致行分布方式的变化。例如，在使用区分大小写的校对规则时，`'and'`在`'Andersen'`之前排序，但在使用不区分大小写的校对规则时，情况则相反。

有关 MySQL 如何处理字符集和校对规则的信息，请参阅第十二章，*字符集、校对规则、Unicode*。

类似地，您可以使用此处显示的`ALTER TABLE`语句使`employees`表按照雇佣员工的年代进行分区，使每一行存储在几个分区中的一个。

```sql
ALTER TABLE employees PARTITION BY RANGE COLUMNS (hired)  (
    PARTITION p0 VALUES LESS THAN ('1970-01-01'),
    PARTITION p1 VALUES LESS THAN ('1980-01-01'),
    PARTITION p2 VALUES LESS THAN ('1990-01-01'),
    PARTITION p3 VALUES LESS THAN ('2000-01-01'),
    PARTITION p4 VALUES LESS THAN ('2010-01-01'),
    PARTITION p5 VALUES LESS THAN (MAXVALUE)
);
```

有关`PARTITION BY RANGE COLUMNS`语法的更多信息，请参阅第 15.1.20 节，“CREATE TABLE 语句”。
