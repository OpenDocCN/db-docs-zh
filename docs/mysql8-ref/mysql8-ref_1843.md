# 26.2.1 范围分区

> 原文：[`dev.mysql.com/doc/refman/8.0/en/partitioning-range.html`](https://dev.mysql.com/doc/refman/8.0/en/partitioning-range.html)

通过范围进行分区的表是这样分区的，即每个分区包含分区表达式值位于给定范围内的行。范围应该是连续的但不重叠，并且使用`VALUES LESS THAN`运算符定义。在接下来的几个示例中，假设你正在创建一个类似以下内容的表，用于保存一家由 1 到 20 号店编号的连锁视频店的人事记录：

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

注意

这里使用的`employees`表没有主键或唯一键。虽然示例在当前讨论的目的上可以正常工作，但你应该记住，在实践中，表极有可能具有主键、唯一键或两者，而用于分区列的可选选择取决于用于这些键的列，如果有的话。有关这些问题的讨论，请参阅 Section 26.6.1, “Partitioning Keys, Primary Keys, and Unique Keys”。

这个表可以根据你的需求以多种方式进行范围分区。一种方法是使用`store_id`列。例如，你可以决定通过添加如下所示的`PARTITION BY RANGE`子句将表分区为 4 部分：

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT NOT NULL,
    store_id INT NOT NULL
)
PARTITION BY RANGE (store_id) (
    PARTITION p0 VALUES LESS THAN (6),
    PARTITION p1 VALUES LESS THAN (11),
    PARTITION p2 VALUES LESS THAN (16),
    PARTITION p3 VALUES LESS THAN (21)
);
```

在这个分区方案中，所有在 1 到 5 号店工作的员工对应的行都存储在`p0`分区中，而在 6 到 10 号店工作的员工对应的行存储在`p1`分区中，依此类推。每个分区按顺序定义，从最低到最高。这是`PARTITION BY RANGE`语法的要求；在这方面，你可以将其类比为 C 或 Java 中一系列`if ... elseif ...`语句。

很容易确定包含数据`(72, 'Mitchell', 'Wilson', '1998-06-25', DEFAULT, 7, 13)`的新行被插入到`p2`分区中，但当你的连锁店增加到第 21 家店时会发生什么？在这个方案下，没有覆盖`store_id`大于 20 的行的规则，因此会导致错误，因为服务器不知道将其放在哪里。你可以通过在`CREATE TABLE`语句中使用“catchall”`VALUES LESS THAN`子句来避免这种情况，提供所有大于明确命名的最高值的值：

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT NOT NULL,
    store_id INT NOT NULL
)
PARTITION BY RANGE (store_id) (
    PARTITION p0 VALUES LESS THAN (6),
    PARTITION p1 VALUES LESS THAN (11),
    PARTITION p2 VALUES LESS THAN (16),
    *PARTITION p3 VALUES LESS THAN MAXVALUE* );
```

（与本章中的其他示例一样，我们假设默认存储引擎是`InnoDB`。）

避免找不到匹配值时出现错误的另一种方法是在`INSERT`语句中使用`IGNORE`关键字。有关示例，请参见 Section 26.2.2, “LIST Partitioning”。

`MAXVALUE`表示一个始终大于最大可能整数值的整数值（在数学语言中，它充当最小上界）。现在，任何`store_id`列值大于或等于 16（定义的最高值）的行都存储在分区`p3`中。在将来的某个时候——当店铺数量增加到 25、30 或更多时，您可以使用`ALTER TABLE`语句为 21-25、26-30 等店铺添加新分区（有关如何执行此操作的详细信息，请参见第 26.3 节，“分区管理”）。

以类似的方式，可以根据员工工作代码对表进行分区，即根据`job_code`列值的范围进行分区。例如——假设两位数的工作代码用于普通（店内）工人，三位数的代码用于办公室和支持人员，四位数的代码用于管理职位——可以使用以下语句创建分区表：

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT NOT NULL,
    store_id INT NOT NULL
)
PARTITION BY RANGE (job_code) (
    PARTITION p0 VALUES LESS THAN (100),
    PARTITION p1 VALUES LESS THAN (1000),
    PARTITION p2 VALUES LESS THAN (10000)
);
```

在这种情况下，所有与店内工人相关的行将存储在分区`p0`中，与办公室和支持人员相关的行将存储在`p1`中，与管理人员相关的行将存储在分区`p2`中。

也可以在`VALUES LESS THAN`子句中使用表达式。但是，MySQL 必须能够将表达式的返回值作为`LESS THAN`（`<`）比较的一部分进行评估。

您可以根据两个`DATE`列中的一个基于表达式来进行分区，而不是根据店铺编号拆分表数据。例如，假设您希望根据每位员工离开公司的年份进行分区；也就是说，基于`YEAR(separated)`的值。下面显示了实现这种分区方案的`CREATE TABLE`语句示例：

```sql
CREATE TABLE employees (
    id INT NOT NULL,
    fname VARCHAR(30),
    lname VARCHAR(30),
    hired DATE NOT NULL DEFAULT '1970-01-01',
    separated DATE NOT NULL DEFAULT '9999-12-31',
    job_code INT,
    store_id INT
)
PARTITION BY RANGE ( YEAR(separated) ) (
    PARTITION p0 VALUES LESS THAN (1991),
    PARTITION p1 VALUES LESS THAN (1996),
    PARTITION p2 VALUES LESS THAN (2001),
    PARTITION p3 VALUES LESS THAN MAXVALUE
);
```

在此方案中，所有在 1991 年之前离开的员工的行存储在分区`p0`中；在 1991 年至 1995 年离开的员工中，存储在`p1`中；在 1996 年至 2000 年离开的员工中，存储在`p2`中；而在 2000 年之后离开的任何工人中，存储在`p3`中。

也可以根据`TIMESTAMP`列的值，使用`UNIX_TIMESTAMP()`函数，基于`RANGE`对表进行分区，如下例所示：

```sql
CREATE TABLE quarterly_report_status (
    report_id INT NOT NULL,
    report_status VARCHAR(20) NOT NULL,
    report_updated TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
)
PARTITION BY RANGE ( UNIX_TIMESTAMP(report_updated) ) (
    PARTITION p0 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-01-01 00:00:00') ),
    PARTITION p1 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-04-01 00:00:00') ),
    PARTITION p2 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-07-01 00:00:00') ),
    PARTITION p3 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-10-01 00:00:00') ),
    PARTITION p4 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-01-01 00:00:00') ),
    PARTITION p5 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-04-01 00:00:00') ),
    PARTITION p6 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-07-01 00:00:00') ),
    PARTITION p7 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-10-01 00:00:00') ),
    PARTITION p8 VALUES LESS THAN ( UNIX_TIMESTAMP('2010-01-01 00:00:00') ),
    PARTITION p9 VALUES LESS THAN (MAXVALUE)
);
```

不允许使用涉及`TIMESTAMP`值的任何其他表达式（请参见 Bug #42849）。

范围分区在以下情况之一或多个情况为真时特别有用：

+   您希望或需要删除“旧”数据。如果您正在使用先前显示的用于 `employees` 表的分区方案，您可以简单地使用 `ALTER TABLE employees DROP PARTITION p0;` 来删除所有在 1991 年之前停止为公司工作的员工的行。（有关更多信息，请参见 第 15.1.9 节“ALTER TABLE 语句” 和 第 26.3 节“分区管理”。）对于行数众多的表，这比运行类似于 `DELETE FROM employees WHERE YEAR(separated) <= 1990;` 的 `DELETE` 查询要高效得多。

+   您希望使用包含日期或时间值的列，或包含从其他系列产生的值。

+   您经常运行依赖于用于对表进行分区的列的查询。例如，当执行类似于 `EXPLAIN SELECT COUNT(*) FROM employees WHERE separated BETWEEN '2000-01-01' AND '2000-12-31' GROUP BY store_id;` 的查询时，MySQL 可以快速确定只需要扫描分区 `p2`，因为其余分区不可能包含任何满足 `WHERE` 子句的记录。有关如何实现这一点的更多信息，请参见 第 26.4 节“分区修剪”。

这种分区的变体是 `RANGE COLUMNS` 分区。通过 `RANGE COLUMNS` 分区，可以使用多个列来定义适用于将行放置在分区中以及确定在执行分区修剪时包含或排除特定分区的分区范围。有关更多信息，请参见 第 26.2.3.1 节“RANGE COLUMNS 分区”。

**基于时间间隔的分区方案。** 如果您希望在 MySQL 8.0 中实现基于时间范围或间隔的分区方案，您有两个选项：

1.  通过 `RANGE` 对表进行分区，并对分区表达式使用在 `DATE`、`TIME` 或 `DATETIME` 列上操作并返回整数值的函数，如下所示：

    ```sql
    CREATE TABLE members (
        firstname VARCHAR(25) NOT NULL,
        lastname VARCHAR(25) NOT NULL,
        username VARCHAR(16) NOT NULL,
        email VARCHAR(35),
        joined DATE NOT NULL
    )
    PARTITION BY RANGE( YEAR(joined) ) (
        PARTITION p0 VALUES LESS THAN (1960),
        PARTITION p1 VALUES LESS THAN (1970),
        PARTITION p2 VALUES LESS THAN (1980),
        PARTITION p3 VALUES LESS THAN (1990),
        PARTITION p4 VALUES LESS THAN MAXVALUE
    );
    ```

    在 MySQL 8.0 中，还可以根据 `TIMESTAMP` 列的值使用 `RANGE` 对表进行分区，使用 `UNIX_TIMESTAMP()` 函数，如下例所示：

    ```sql
    CREATE TABLE quarterly_report_status (
        report_id INT NOT NULL,
        report_status VARCHAR(20) NOT NULL,
        report_updated TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
    )
    PARTITION BY RANGE ( UNIX_TIMESTAMP(report_updated) ) (
        PARTITION p0 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-01-01 00:00:00') ),
        PARTITION p1 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-04-01 00:00:00') ),
        PARTITION p2 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-07-01 00:00:00') ),
        PARTITION p3 VALUES LESS THAN ( UNIX_TIMESTAMP('2008-10-01 00:00:00') ),
        PARTITION p4 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-01-01 00:00:00') ),
        PARTITION p5 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-04-01 00:00:00') ),
        PARTITION p6 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-07-01 00:00:00') ),
        PARTITION p7 VALUES LESS THAN ( UNIX_TIMESTAMP('2009-10-01 00:00:00') ),
        PARTITION p8 VALUES LESS THAN ( UNIX_TIMESTAMP('2010-01-01 00:00:00') ),
        PARTITION p9 VALUES LESS THAN (MAXVALUE)
    );
    ```

    在 MySQL 8.0 中，不允许使用涉及 `TIMESTAMP` 值的任何其他表达式。（参见 Bug #42849。）

    注意

    在 MySQL 8.0 中，也可以使用`UNIX_TIMESTAMP(timestamp_column)`作为按`LIST`分区的表的分区表达式。然而，通常不太实用。

1.  通过`RANGE COLUMNS`按`DATE`或`DATETIME`列作为分区列对表进行分区。例如，`members`表可以直接使用`joined`列定义，如下所示：

    ```sql
    CREATE TABLE members (
        firstname VARCHAR(25) NOT NULL,
        lastname VARCHAR(25) NOT NULL,
        username VARCHAR(16) NOT NULL,
        email VARCHAR(35),
        joined DATE NOT NULL
    )
    PARTITION BY RANGE COLUMNS(joined) (
        PARTITION p0 VALUES LESS THAN ('1960-01-01'),
        PARTITION p1 VALUES LESS THAN ('1970-01-01'),
        PARTITION p2 VALUES LESS THAN ('1980-01-01'),
        PARTITION p3 VALUES LESS THAN ('1990-01-01'),
        PARTITION p4 VALUES LESS THAN MAXVALUE
    );
    ```

注意

使用日期或时间类型的分区列，而不是`DATE`或`DATETIME`，在`RANGE COLUMNS`中不受支持。
