# 26.3.1 范围和列表分区的管理

> 原文：[`dev.mysql.com/doc/refman/8.0/en/partitioning-management-range-list.html`](https://dev.mysql.com/doc/refman/8.0/en/partitioning-management-range-list.html)

添加和删除范围和列表分区的操作方式类似，因此我们在本节讨论了这两种分区管理方式。 有关处理按哈希或键分区的表的信息，请参阅 第 26.3.2 节，“HASH 和 KEY 分区的管理”。

通过使用带有 `DROP PARTITION` 选项的 `ALTER TABLE` 语句，可以删除按 `RANGE` 或 `LIST` 分区的表中的分区。 假设您已创建了一个按范围分区的表，然后使用以下 `CREATE TABLE` 和 `INSERT` 语句插入了 10 条记录：

```sql
mysql> CREATE TABLE tr (id INT, name VARCHAR(50), purchased DATE)
 ->     PARTITION BY RANGE( YEAR(purchased) ) (
 ->         PARTITION p0 VALUES LESS THAN (1990),
 ->         PARTITION p1 VALUES LESS THAN (1995),
 ->         PARTITION p2 VALUES LESS THAN (2000),
 ->         PARTITION p3 VALUES LESS THAN (2005),
 ->         PARTITION p4 VALUES LESS THAN (2010),
 ->         PARTITION p5 VALUES LESS THAN (2015)
 ->     );
Query OK, 0 rows affected (0.28 sec)

mysql> INSERT INTO tr VALUES
 ->     (1, 'desk organiser', '2003-10-15'),
 ->     (2, 'alarm clock', '1997-11-05'),
 ->     (3, 'chair', '2009-03-10'),
 ->     (4, 'bookcase', '1989-01-10'),
 ->     (5, 'exercise bike', '2014-05-09'),
 ->     (6, 'sofa', '1987-06-05'),
 ->     (7, 'espresso maker', '2011-11-22'),
 ->     (8, 'aquarium', '1992-08-04'),
 ->     (9, 'study desk', '2006-09-16'),
 ->     (10, 'lava lamp', '1998-12-25');
Query OK, 10 rows affected (0.05 sec)
Records: 10  Duplicates: 0  Warnings: 0
```

您可以查看应该插入到分区 `p2` 中的项目，如下所示：

```sql
mysql> SELECT * FROM tr
 ->     WHERE purchased BETWEEN '1995-01-01' AND '1999-12-31';
+------+-------------+------------+
| id   | name        | purchased  |
+------+-------------+------------+
|    2 | alarm clock | 1997-11-05 |
|   10 | lava lamp   | 1998-12-25 |
+------+-------------+------------+
2 rows in set (0.00 sec)
```

您还可以使用分区选择获取此信息，如下所示：

```sql
mysql> SELECT * FROM tr PARTITION (p2);
+------+-------------+------------+
| id   | name        | purchased  |
+------+-------------+------------+
|    2 | alarm clock | 1997-11-05 |
|   10 | lava lamp   | 1998-12-25 |
+------+-------------+------------+
2 rows in set (0.00 sec)
```

更多信息请参见 第 26.5 节，“分区选择”。

要删除名为 `p2` 的分区，请执行以下命令：

```sql
mysql> ALTER TABLE tr DROP PARTITION p2;
Query OK, 0 rows affected (0.03 sec)
```

注意

`NDBCLUSTER` 存储引擎不支持 `ALTER TABLE ... DROP PARTITION`。 但是，它支持本章中描述的与分区相关的其他 `ALTER TABLE` 扩展。

非常重要的一点是，*当您删除一个分区时，也会删除存储在该分区中的所有数据*。 通过重新运行先前的 `SELECT` 查询，您可以看到这一点：

```sql
mysql> SELECT * FROM tr WHERE purchased
 -> BETWEEN '1995-01-01' AND '1999-12-31';
Empty set (0.00 sec)
```

注意

`DROP PARTITION` 受本地分区就地 API 支持，可与 `ALGORITHM={COPY|INPLACE}` 一起使用。 使用 `ALGORITHM=INPLACE` 的 `DROP PARTITION` 删除存储在分区中的数据并删除该分区。 但是，使用 `ALGORITHM=COPY` 或 `old_alter_table=ON` 的 `DROP PARTITION` 会重建分区表，并尝试将无法移动到另一个具有兼容 `PARTITION ... VALUES` 定义的分区的数据移动到另一个分区。 无法移动到另一个分区的数据将被删除。

因此，在执行 `ALTER TABLE ... DROP PARTITION` 之前，您必须具有表的 `DROP` 权限。

如果您希望删除所有分区中的所有数据，同时保留表定义及其分区方案，请使用`TRUNCATE TABLE`语句。（请参阅第 15.1.37 节，“TRUNCATE TABLE Statement”。）

如果您打算更改表的分区而*不*丢失数据，请使用`ALTER TABLE ... REORGANIZE PARTITION`。有关`REORGANIZE PARTITION`的信息，请参见下文或第 15.1.9 节，“ALTER TABLE Statement”。

如果现在执行`SHOW CREATE TABLE`语句，您可以看到表的分区结构已经发生了变化：

```sql
mysql> SHOW CREATE TABLE tr\G
*************************** 1\. row ***************************
       Table: tr
Create Table: CREATE TABLE `tr` (
  `id` int(11) DEFAULT NULL,
  `name` varchar(50) DEFAULT NULL,
  `purchased` date DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
/*!50100 PARTITION BY RANGE ( YEAR(purchased))
(PARTITION p0 VALUES LESS THAN (1990) ENGINE = InnoDB,
 PARTITION p1 VALUES LESS THAN (1995) ENGINE = InnoDB,
 PARTITION p3 VALUES LESS THAN (2005) ENGINE = InnoDB,
 PARTITION p4 VALUES LESS THAN (2010) ENGINE = InnoDB,
 PARTITION p5 VALUES LESS THAN (2015) ENGINE = InnoDB) */ 1 row in set (0.00 sec)
```

当你在更改后的表中插入具有`purchased`列值在`'1995-01-01'`和`'2004-12-31'`之间（包括这两个日期）的新行时，这些行将存储在分区`p3`中。您可以按照以下步骤验证这一点：

```sql
mysql> INSERT INTO tr VALUES (11, 'pencil holder', '1995-07-12');
Query OK, 1 row affected (0.00 sec)

mysql> SELECT * FROM tr WHERE purchased
 -> BETWEEN '1995-01-01' AND '2004-12-31';
+------+----------------+------------+
| id   | name           | purchased  |
+------+----------------+------------+
|    1 | desk organiser | 2003-10-15 |
|   11 | pencil holder  | 1995-07-12 |
+------+----------------+------------+
2 rows in set (0.00 sec)

mysql> ALTER TABLE tr DROP PARTITION p3;
Query OK, 0 rows affected (0.03 sec)

mysql> SELECT * FROM tr WHERE purchased
 -> BETWEEN '1995-01-01' AND '2004-12-31';
Empty set (0.00 sec)
```

由于服务器不会像等效的`DELETE`查询那样报告由于`ALTER TABLE ... DROP PARTITION`而从表中删除的行数。

删除`LIST`分区与删除`RANGE`分区使用完全相同的`ALTER TABLE ... DROP PARTITION`语法。然而，这对之后对表的使用有一个重要的区别：您不能再向表中插入具有被删除分区定义的值列表中的任何值的行。（请参阅第 26.2.2 节，“LIST 分区”，以获取示例。）

要向先前分区的表添加新的范围或列表分区，请使用`ALTER TABLE ... ADD PARTITION`语句。对于按`RANGE`分区的表，这可以用于在现有分区列表的末尾添加新的范围。假设您有一个包含组织成员数据的分区表，其定义如下：

```sql
CREATE TABLE members (
    id INT,
    fname VARCHAR(25),
    lname VARCHAR(25),
    dob DATE
)
PARTITION BY RANGE( YEAR(dob) ) (
    PARTITION p0 VALUES LESS THAN (1980),
    PARTITION p1 VALUES LESS THAN (1990),
    PARTITION p2 VALUES LESS THAN (2000)
);
```

进一步假设成员的最小年龄为 16 岁。随着日历接近 2015 年底，您意识到必须很快准备好接纳 2000 年（及以后）出生的成员。您可以修改`members`表以适应在 2000 年至 2010 年出生的新成员，如下所示：

```sql
ALTER TABLE members ADD PARTITION (PARTITION p3 VALUES LESS THAN (2010));
```

对于按范围分区的表，您可以使用`ADD PARTITION`仅向分区列表的高端添加新分区。尝试以这种方式在现有分区之间或之前添加新分区会导致错误，如下所示：

```sql
mysql> ALTER TABLE members
     >     ADD PARTITION (
     >     PARTITION n VALUES LESS THAN (1970));
ERROR 1463 (HY000): VALUES LESS THAN value must be strictly »
   increasing for each partition
```

您可以通过将第一个分区重新组织为两个新分区，将它们之间的范围分割，来解决这个问题，就像这样：

```sql
ALTER TABLE members
    REORGANIZE PARTITION p0 INTO (
        PARTITION n0 VALUES LESS THAN (1970),
        PARTITION n1 VALUES LESS THAN (1980)
);
```

使用`SHOW CREATE TABLE`，您可以看到`ALTER TABLE`语句已经产生了预期的效果：

```sql
mysql> SHOW CREATE TABLE members\G
*************************** 1\. row ***************************
       Table: members
Create Table: CREATE TABLE `members` (
  `id` int(11) DEFAULT NULL,
  `fname` varchar(25) DEFAULT NULL,
  `lname` varchar(25) DEFAULT NULL,
  `dob` date DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
/*!50100 PARTITION BY RANGE ( YEAR(dob))
(PARTITION n0 VALUES LESS THAN (1970) ENGINE = InnoDB,
 PARTITION n1 VALUES LESS THAN (1980) ENGINE = InnoDB,
 PARTITION p1 VALUES LESS THAN (1990) ENGINE = InnoDB,
 PARTITION p2 VALUES LESS THAN (2000) ENGINE = InnoDB,
 PARTITION p3 VALUES LESS THAN (2010) ENGINE = InnoDB) */ 1 row in set (0.00 sec)
```

另请参阅 Section 15.1.9.1, “ALTER TABLE Partition Operations”。

您还可以使用`ALTER TABLE ... ADD PARTITION`来向由`LIST`分区的表中添加新分区。假设一个表`tt`是使用以下`CREATE TABLE`语句定义的：

```sql
CREATE TABLE tt (
    id INT,
    data INT
)
PARTITION BY LIST(data) (
    PARTITION p0 VALUES IN (5, 10, 15),
    PARTITION p1 VALUES IN (6, 12, 18)
);
```

您可以添加一个新分区，用于存储具有`data`列值`7`、`14`和`21`的行，如下所示：

```sql
ALTER TABLE tt ADD PARTITION (PARTITION p2 VALUES IN (7, 14, 21));
```

请记住，*不能*添加一个新的`LIST`分区，其中包含已经包含在现有分区值列表中的任何值。如果尝试这样做，将导致错误：

```sql
mysql> ALTER TABLE tt ADD PARTITION 
     >     (PARTITION np VALUES IN (4, 8, 12));
ERROR 1465 (HY000): Multiple definition of same constant »
                    in list partitioning
```

因为具有`data`列值`12`的任何行已经分配给了分区`p1`，所以无法在表`tt`上创建一个包含`12`在其值列表中的新分区。为了实现这一点，您可以删除`p1`，然后添加`np`，然后一个新的`p1`，并修改定义。然而，正如前面讨论的，这将导致所有存储在`p1`中的数据丢失，而且通常情况下这并不是您真正想要做的。另一个解决方案可能是制作一个具有新分区的表的副本，并使用`CREATE TABLE ... SELECT ...`将数据复制到其中，然后删除旧表并重命名新表，但是在处理大量数据时可能非常耗时。在需要高可用性的情况下，这也可能不可行。

您可以在单个`ALTER TABLE ... ADD PARTITION`语句中添加多个分区，如下所示：

```sql
CREATE TABLE employees (
  id INT NOT NULL,
  fname VARCHAR(50) NOT NULL,
  lname VARCHAR(50) NOT NULL,
  hired DATE NOT NULL
)
PARTITION BY RANGE( YEAR(hired) ) (
  PARTITION p1 VALUES LESS THAN (1991),
  PARTITION p2 VALUES LESS THAN (1996),
  PARTITION p3 VALUES LESS THAN (2001),
  PARTITION p4 VALUES LESS THAN (2005)
);

ALTER TABLE employees ADD PARTITION (
    PARTITION p5 VALUES LESS THAN (2010),
    PARTITION p6 VALUES LESS THAN MAXVALUE
);
```

幸运的是，MySQL 的分区实现提供了重新定义分区而不丢失数据的方法。让我们首先看一下涉及`RANGE`分区的几个简单示例。回想一下现在定义如下的`members`表：

```sql
mysql> SHOW CREATE TABLE members\G
*************************** 1\. row ***************************
       Table: members
Create Table: CREATE TABLE `members` (
  `id` int(11) DEFAULT NULL,
  `fname` varchar(25) DEFAULT NULL,
  `lname` varchar(25) DEFAULT NULL,
  `dob` date DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
/*!50100 PARTITION BY RANGE ( YEAR(dob))
(PARTITION n0 VALUES LESS THAN (1970) ENGINE = InnoDB,
 PARTITION n1 VALUES LESS THAN (1980) ENGINE = InnoDB,
 PARTITION p1 VALUES LESS THAN (1990) ENGINE = InnoDB,
 PARTITION p2 VALUES LESS THAN (2000) ENGINE = InnoDB,
 PARTITION p3 VALUES LESS THAN (2010) ENGINE = InnoDB) */ 1 row in set (0.00 sec)
```

假设您想将所有出生在 1960 年之前的会员行移动到一个单独的分区中。正如我们已经看到的，这不能通过`ALTER TABLE ... ADD PARTITION`来实现。然而，您可以使用另一个与分区相关的扩展来`ALTER TABLE`来完成这个任务：

```sql
ALTER TABLE members REORGANIZE PARTITION n0 INTO (
    PARTITION s0 VALUES LESS THAN (1960),
    PARTITION s1 VALUES LESS THAN (1970)
);
```

实际上，这个命令将分区`p0`分割成两个新分区`s0`和`s1`。它还根据两个`PARTITION ... VALUES ...`子句中体现的规则，将存储在`p0`中的数据移动到新分区中，因此`s0`只包含那些`YEAR(dob)`小于 1960 的记录，而`s1`包含那些`YEAR(dob)`大于或等于 1960 但小于 1970 的行。

`REORGANIZE PARTITION`子句也可用于合并相邻分区。您可以撤销对`members`表的上一个语句的影响，如下所示：

```sql
ALTER TABLE members REORGANIZE PARTITION s0,s1 INTO (
    PARTITION p0 VALUES LESS THAN (1970)
);
```

使用`REORGANIZE PARTITION`拆分或合并分区时不会丢失任何数据。在执行上述语句时，MySQL 将所有存储在分区`s0`和`s1`中的记录移动到分区`p0`中。

`REORGANIZE PARTITION`的一般语法如下所示：

```sql
ALTER TABLE *tbl_name*
    REORGANIZE PARTITION *partition_list*
    INTO (*partition_definitions*);
```

这里，*`tbl_name`*是分区表的名称，*`partition_list`*是要更改的一个或多个现有分区的名称的逗号分隔列表。*`partition_definitions`*是一个逗号分隔的新分区定义列表，遵循与`CREATE TABLE`中使用的*`partition_definitions`*列表相同的规则。在使用`REORGANIZE PARTITION`时，您不限于将多个分区合并为一个，或将一个分区分割为多个。例如，您可以将`members`表的四个分区重新组织为两个，如下所示：

```sql
ALTER TABLE members REORGANIZE PARTITION p0,p1,p2,p3 INTO (
    PARTITION m0 VALUES LESS THAN (1980),
    PARTITION m1 VALUES LESS THAN (2000)
);
```

您还可以在按`LIST`进行分区的表上使用`REORGANIZE PARTITION`。让我们回到向列表分区的`tt`表添加新分区的问题，并因为新分区的值已经存在于现有分区的值列表中而失败。我们可以通过添加一个仅包含非冲突值的分区，然后重新组织新分区和现有分区，使存储在现有分区中的值现在移动到新分区来处理这个问题：

```sql
ALTER TABLE tt ADD PARTITION (PARTITION np VALUES IN (4, 8));
ALTER TABLE tt REORGANIZE PARTITION p1,np INTO (
    PARTITION p1 VALUES IN (6, 18),
    PARTITION np VALUES in (4, 8, 12)
);
```

在使用`ALTER TABLE ... REORGANIZE PARTITION`重新分区按`RANGE`或`LIST`进行分区的表时，请记住以下要点：

+   用于确定新分区方案的`PARTITION`选项受与`CREATE TABLE`语句相同的规则约束。

    新的`RANGE`分区方案不能有任何重叠的范围；新的`LIST`分区方案不能有任何重叠的值集。

+   *`partition_definitions`*列表中的分区组合应该总体上与*`partition_list`*中命名的组合分区涵盖相同的范围或值集。

    例如，在本节示例中使用的`members`表中，分区`p1`和`p2`一起涵盖 1980 年至 1999 年的年份。对这两个分区的任何重新组织应该总体上涵盖相同的年份范围。

+   对于按`RANGE`进行分区的表，您只能重新组织相邻分区；您不能跳过范围分区。

    例如，您不能使用以`ALTER TABLE members REORGANIZE PARTITION p0,p2 INTO ...`开头的语句重新组织示例`members`表，因为`p0`涵盖 1970 年之前的年份，而`p2`涵盖 1990 年至 1999 年的年份，因此这些不是相邻的分区。（在这种情况下，您不能跳过分区`p1`。）

+   你不能使用`REORGANIZE PARTITION`来改变表使用的分区类型（例如，你不能将`RANGE`分区更改为`HASH`分区或反之）。你也不能使用这个语句来更改分区表达式或列。要完成这两项任务而不必删除和重新创建表，你可以使用`ALTER TABLE ... PARTITION BY ...`，如下所示：

    ```sql
    ALTER TABLE members
        PARTITION BY HASH( YEAR(dob) )
        PARTITIONS 8;
    ```
