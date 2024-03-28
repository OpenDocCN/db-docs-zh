# 26.5 分区选择

> 原文：[`dev.mysql.com/doc/refman/8.0/en/partitioning-selection.html`](https://dev.mysql.com/doc/refman/8.0/en/partitioning-selection.html)

支持为匹配给定`WHERE`条件的行显式选择分区和子分区。分区选择类似于分区修剪，只检查特定分区是否匹配，但在两个关键方面有所不同：

1.  要检查的分区由语句的发出者指定，与自动分区修剪不同。

1.  分区修剪仅适用于查询，而显式选择分区支持查询和一些 DML 语句。

支持显式分区选择的 SQL 语句列在此处：

+   `SELECT`

+   `DELETE`

+   `INSERT`

+   `REPLACE`

+   `UPDATE`

+   `LOAD DATA`。

+   `LOAD XML`。

本节的其余部分讨论了显式分区选择，通常适用于刚刚列出的语句，并提供了一些示例。

显式分区选择是使用`PARTITION`选项实现的。对于所有支持的语句，此选项使用以下语法：

```sql
 PARTITION (*partition_names*)

      *partition_names*:
          *partition_name*, ...
```

此选项始终跟随分区或子分区所属的表的名称。*`partition_names`*是要使用的分区或子分区的逗号分隔列表。此列表中的每个名称必须是指定表的现有分区或子分区的名称；如果找不到任何分区或子分区，则该语句将失败并显示错误（分区'*`partition_name`*'不存在）。*`partition_names`*中列出的分区和子分区可以以任何顺序列出，并且可以重叠。

使用`PARTITION`选项时，仅检查列出的分区和子分区以匹配行。此选项可用于`SELECT`语句，以确定哪些行属于给定分区。考虑一个名为`employees`的分区表，使用以下语句创建和填充：

```sql
SET @@SQL_MODE = '';

CREATE TABLE employees  (
    id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    fname VARCHAR(25) NOT NULL,
    lname VARCHAR(25) NOT NULL,
    store_id INT NOT NULL,
    department_id INT NOT NULL
)
    PARTITION BY RANGE(id)  (
        PARTITION p0 VALUES LESS THAN (5),
        PARTITION p1 VALUES LESS THAN (10),
        PARTITION p2 VALUES LESS THAN (15),
        PARTITION p3 VALUES LESS THAN MAXVALUE
);

INSERT INTO employees VALUES
    ('', 'Bob', 'Taylor', 3, 2), ('', 'Frank', 'Williams', 1, 2),
    ('', 'Ellen', 'Johnson', 3, 4), ('', 'Jim', 'Smith', 2, 4),
    ('', 'Mary', 'Jones', 1, 1), ('', 'Linda', 'Black', 2, 3),
    ('', 'Ed', 'Jones', 2, 1), ('', 'June', 'Wilson', 3, 1),
    ('', 'Andy', 'Smith', 1, 3), ('', 'Lou', 'Waters', 2, 4),
    ('', 'Jill', 'Stone', 1, 4), ('', 'Roger', 'White', 3, 2),
    ('', 'Howard', 'Andrews', 1, 2), ('', 'Fred', 'Goldberg', 3, 3),
    ('', 'Barbara', 'Brown', 2, 3), ('', 'Alice', 'Rogers', 2, 2),
    ('', 'Mark', 'Morgan', 3, 3), ('', 'Karen', 'Cole', 3, 2);
```

您可以这样查看存储在分区`p1`中的行：

```sql
mysql> SELECT * FROM employees PARTITION (p1);
+----+-------+--------+----------+---------------+
| id | fname | lname  | store_id | department_id |
+----+-------+--------+----------+---------------+
|  5 | Mary  | Jones  |        1 |             1 |
|  6 | Linda | Black  |        2 |             3 |
|  7 | Ed    | Jones  |        2 |             1 |
|  8 | June  | Wilson |        3 |             1 |
|  9 | Andy  | Smith  |        1 |             3 |
+----+-------+--------+----------+---------------+
5 rows in set (0.00 sec)
```

结果与通过查询`SELECT * FROM employees WHERE id BETWEEN 5 AND 9`获得的结果相同。

要从多个分区获取行，请将它们的名称作为逗号分隔的列表提供。例如，`SELECT * FROM employees PARTITION (p1, p2)`返回分区`p1`和`p2`中的所有行，同时排除其余分区的行。

对分区表的任何有效查询都可以通过`PARTITION`选项重写，以限制结果为一个或多个所需的分区。您可以使用`WHERE`条件、`ORDER BY`和`LIMIT`选项等。您还可以使用带有`HAVING`和`GROUP BY`选项的聚合函数。在先前定义的`employees`表上运行以下每个查询都会产生有效结果：

```sql
mysql> SELECT * FROM employees PARTITION (p0, p2)
 ->     WHERE lname LIKE 'S%';
+----+-------+-------+----------+---------------+
| id | fname | lname | store_id | department_id |
+----+-------+-------+----------+---------------+
|  4 | Jim   | Smith |        2 |             4 |
| 11 | Jill  | Stone |        1 |             4 |
+----+-------+-------+----------+---------------+
2 rows in set (0.00 sec)

mysql> SELECT id, CONCAT(fname, ' ', lname) AS name
 ->     FROM employees PARTITION (p0) ORDER BY lname;
+----+----------------+
| id | name           |
+----+----------------+
|  3 | Ellen Johnson  |
|  4 | Jim Smith      |
|  1 | Bob Taylor     |
|  2 | Frank Williams |
+----+----------------+
4 rows in set (0.06 sec)

mysql> SELECT store_id, COUNT(department_id) AS c
 ->     FROM employees PARTITION (p1,p2,p3)
 ->     GROUP BY store_id HAVING c > 4;
+---+----------+
| c | store_id |
+---+----------+
| 5 |        2 |
| 5 |        3 |
+---+----------+
2 rows in set (0.00 sec)
```

使用分区选择的语句可以与使用任何支持的分区类型的表一起使用。当使用`[LINEAR] HASH`或`[LINEAR] KEY`分区创建表时，如果未指定分区的名称，MySQL 会自动将分区命名为`p0`、`p1`、`p2`、...、`p*`N-1`*`，其中*`N`*是分区的数量。对于未明确命名的子分区，MySQL 会自动为每个分区`p*`X`*`中的子分区分配名称`p*`X`*sp0`、`p*`X`*sp1`、`p*`X`*sp2`、...、`p*`X`*sp*`M-1`*`，其中*`M`*是子分区的数量。在针对这个表执行`SELECT`（或其他允许显式分区选择的 SQL 语句）时，您可以在`PARTITION`选项中使用这些生成的名称，如下所示：

```sql
mysql> CREATE TABLE employees_sub  (
 ->     id INT NOT NULL AUTO_INCREMENT,
 ->     fname VARCHAR(25) NOT NULL,
 ->     lname VARCHAR(25) NOT NULL,
 ->     store_id INT NOT NULL,
 ->     department_id INT NOT NULL,
 ->     PRIMARY KEY pk (id, lname)
 -> )
 ->     PARTITION BY RANGE(id)
 ->     SUBPARTITION BY KEY (lname)
 ->     SUBPARTITIONS 2 (
 ->         PARTITION p0 VALUES LESS THAN (5),
 ->         PARTITION p1 VALUES LESS THAN (10),
 ->         PARTITION p2 VALUES LESS THAN (15),
 ->         PARTITION p3 VALUES LESS THAN MAXVALUE
 -> );
Query OK, 0 rows affected (1.14 sec)

mysql> INSERT INTO employees_sub   # reuse data in employees table
 ->     SELECT * FROM employees;
Query OK, 18 rows affected (0.09 sec)
Records: 18  Duplicates: 0  Warnings: 0

mysql> SELECT id, CONCAT(fname, ' ', lname) AS name
 ->     FROM employees_sub PARTITION (p2sp1);
+----+---------------+
| id | name          |
+----+---------------+
| 10 | Lou Waters    |
| 14 | Fred Goldberg |
+----+---------------+
2 rows in set (0.00 sec)
```

您还可以在`SELECT`部分的`INSERT ... SELECT`语句中使用`PARTITION`选项，如下所示：

```sql
mysql> CREATE TABLE employees_copy LIKE employees;
Query OK, 0 rows affected (0.28 sec)

mysql> INSERT INTO employees_copy
 ->     SELECT * FROM employees PARTITION (p2);
Query OK, 5 rows affected (0.04 sec)
Records: 5  Duplicates: 0  Warnings: 0

mysql> SELECT * FROM employees_copy;
+----+--------+----------+----------+---------------+
| id | fname  | lname    | store_id | department_id |
+----+--------+----------+----------+---------------+
| 10 | Lou    | Waters   |        2 |             4 |
| 11 | Jill   | Stone    |        1 |             4 |
| 12 | Roger  | White    |        3 |             2 |
| 13 | Howard | Andrews  |        1 |             2 |
| 14 | Fred   | Goldberg |        3 |             3 |
+----+--------+----------+----------+---------------+
5 rows in set (0.00 sec)
```

分区选择也可以与连接一起使用。假设我们使用以下语句创建和填充了两个表：

```sql
CREATE TABLE stores (
    id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    city VARCHAR(30) NOT NULL
)
    PARTITION BY HASH(id)
    PARTITIONS 2;

INSERT INTO stores VALUES
    ('', 'Nambucca'), ('', 'Uranga'),
    ('', 'Bellingen'), ('', 'Grafton');

CREATE TABLE departments  (
    id INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(30) NOT NULL
)
    PARTITION BY KEY(id)
    PARTITIONS 2;

INSERT INTO departments VALUES
    ('', 'Sales'), ('', 'Customer Service'),
    ('', 'Delivery'), ('', 'Accounting');
```

您可以明确地从连接中的任何或所有表中选择分区（或子分区，或两者）。（用于从给定表中选择分区的`PARTITION`选项紧随表名之后，位于所有其他选项之前，包括任何表别名。）例如，以下查询获取所有在销售或交付部门（`departments`表的分区`p1`）中的商店（`stores`表的分区`p0`）中工作的员工的姓名、员工 ID、部门和城市：

```sql
mysql> SELECT
 ->     e.id AS 'Employee ID', CONCAT(e.fname, ' ', e.lname) AS Name,
 ->     s.city AS City, d.name AS department
 -> FROM employees AS e
 ->     JOIN stores PARTITION (p1) AS s ON e.store_id=s.id
 ->     JOIN departments PARTITION (p0) AS d ON e.department_id=d.id
 -> ORDER BY e.lname;
+-------------+---------------+-----------+------------+
| Employee ID | Name          | City      | department |
+-------------+---------------+-----------+------------+
|          14 | Fred Goldberg | Bellingen | Delivery   |
|           5 | Mary Jones    | Nambucca  | Sales      |
|          17 | Mark Morgan   | Bellingen | Delivery   |
|           9 | Andy Smith    | Nambucca  | Delivery   |
|           8 | June Wilson   | Bellingen | Sales      |
+-------------+---------------+-----------+------------+
5 rows in set (0.00 sec)
```

有关 MySQL 中连接的一般信息，请参见第 15.2.13.2 节，“JOIN Clause”。

当`PARTITION`选项与`DELETE`语句一起使用时，只有在选项中列出的那些分区（和子分区，如果有）会被检查以删除行。任何其他分区都会被忽略，如下所示：

```sql
mysql> SELECT * FROM employees WHERE fname LIKE 'j%';
+----+-------+--------+----------+---------------+
| id | fname | lname  | store_id | department_id |
+----+-------+--------+----------+---------------+
|  4 | Jim   | Smith  |        2 |             4 |
|  8 | June  | Wilson |        3 |             1 |
| 11 | Jill  | Stone  |        1 |             4 |
+----+-------+--------+----------+---------------+
3 rows in set (0.00 sec)

mysql> DELETE FROM employees PARTITION (p0, p1)
 ->     WHERE fname LIKE 'j%';
Query OK, 2 rows affected (0.09 sec)

mysql> SELECT * FROM employees WHERE fname LIKE 'j%';
+----+-------+-------+----------+---------------+
| id | fname | lname | store_id | department_id |
+----+-------+-------+----------+---------------+
| 11 | Jill  | Stone |        1 |             4 |
+----+-------+-------+----------+---------------+
1 row in set (0.00 sec)
```

只有与`WHERE`条件匹配的`p0`和`p1`分区中的两行被删除。从第二次运行`SELECT`时的结果中可以看到，表中仍然存在一行与`WHERE`条件匹配，但位于不同的分区（`p2`）。

使用显式分区选择的`UPDATE`语句的行为相同；只有在确定要更新的行时，才考虑`PARTITION`选项引用的分区中的行，可以通过执行以下语句来查看：

```sql
mysql> UPDATE employees PARTITION (p0) 
 ->     SET store_id = 2 WHERE fname = 'Jill';
Query OK, 0 rows affected (0.00 sec)
Rows matched: 0  Changed: 0  Warnings: 0

mysql> SELECT * FROM employees WHERE fname = 'Jill';
+----+-------+-------+----------+---------------+
| id | fname | lname | store_id | department_id |
+----+-------+-------+----------+---------------+
| 11 | Jill  | Stone |        1 |             4 |
+----+-------+-------+----------+---------------+
1 row in set (0.00 sec)

mysql> UPDATE employees PARTITION (p2)
 ->     SET store_id = 2 WHERE fname = 'Jill';
Query OK, 1 row affected (0.09 sec)
Rows matched: 1  Changed: 1  Warnings: 0

mysql> SELECT * FROM employees WHERE fname = 'Jill';
+----+-------+-------+----------+---------------+
| id | fname | lname | store_id | department_id |
+----+-------+-------+----------+---------------+
| 11 | Jill  | Stone |        2 |             4 |
+----+-------+-------+----------+---------------+
1 row in set (0.00 sec)
```

同样，当与`DELETE`一起使用`PARTITION`时，只有在分区列表中命名的分区中的行才会被检查删除。

对于插入行的语句，行为有所不同，找不到合适分区会导致语句失败。这对于`INSERT`和`REPLACE`语句都是适用的，如下所示：

```sql
mysql> INSERT INTO employees PARTITION (p2) VALUES (20, 'Jan', 'Jones', 1, 3);
ERROR 1729 (HY000): Found a row not matching the given partition set
mysql> INSERT INTO employees PARTITION (p3) VALUES (20, 'Jan', 'Jones', 1, 3);
Query OK, 1 row affected (0.07 sec)

mysql> REPLACE INTO employees PARTITION (p0) VALUES (20, 'Jan', 'Jones', 3, 2);
ERROR 1729 (HY000): Found a row not matching the given partition set 
mysql> REPLACE INTO employees PARTITION (p3) VALUES (20, 'Jan', 'Jones', 3, 2);
Query OK, 2 rows affected (0.09 sec)
```

对于使用`InnoDB`存储引擎的分区表写入多行的语句：如果`VALUES`后面的列表中的任何行无法写入到*`partition_names`*列表中指定的分区之一，则整个语句将失败，不会写入任何行。这在下面的示例中展示了`INSERT`语句，重用了之前创建的`employees`表：

```sql
mysql> ALTER TABLE employees
 ->     REORGANIZE PARTITION p3 INTO (
 ->         PARTITION p3 VALUES LESS THAN (20),
 ->         PARTITION p4 VALUES LESS THAN (25),
 ->         PARTITION p5 VALUES LESS THAN MAXVALUE
 ->     );
Query OK, 6 rows affected (2.09 sec)
Records: 6  Duplicates: 0  Warnings: 0

mysql> SHOW CREATE TABLE employees\G
*************************** 1\. row ***************************
       Table: employees
Create Table: CREATE TABLE `employees` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `fname` varchar(25) NOT NULL,
  `lname` varchar(25) NOT NULL,
  `store_id` int(11) NOT NULL,
  `department_id` int(11) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=27 DEFAULT CHARSET=utf8mb4
/*!50100 PARTITION BY RANGE (id)
(PARTITION p0 VALUES LESS THAN (5) ENGINE = InnoDB,
 PARTITION p1 VALUES LESS THAN (10) ENGINE = InnoDB,
 PARTITION p2 VALUES LESS THAN (15) ENGINE = InnoDB,
 PARTITION p3 VALUES LESS THAN (20) ENGINE = InnoDB,
 PARTITION p4 VALUES LESS THAN (25) ENGINE = InnoDB,
 PARTITION p5 VALUES LESS THAN MAXVALUE ENGINE = InnoDB) */ 1 row in set (0.00 sec)

mysql> INSERT INTO employees PARTITION (p3, p4) VALUES
 ->     (24, 'Tim', 'Greene', 3, 1),  (26, 'Linda', 'Mills', 2, 1);
ERROR 1729 (HY000): Found a row not matching the given partition set 
mysql> INSERT INTO employees PARTITION (p3, p4, p5) VALUES
 ->     (24, 'Tim', 'Greene', 3, 1),  (26, 'Linda', 'Mills', 2, 1);
Query OK, 2 rows affected (0.06 sec)
Records: 2  Duplicates: 0  Warnings: 0
```

对于写入多行的`INSERT`语句和`REPLACE`语句，上述内容都是适用的。

对于使用提供自动分区的存储引擎（如`NDB`）的表，分区选择被禁用。
