# 26.2 分区类型

> 原文：[`dev.mysql.com/doc/refman/8.0/en/partitioning-types.html`](https://dev.mysql.com/doc/refman/8.0/en/partitioning-types.html)

26.2.1 范围分区

26.2.2 LIST 分区

26.2.3 列分区

26.2.4 HASH 分区

26.2.5 KEY 分区

26.2.6 子分区

26.2.7 MySQL 分区如何处理 NULL 值

本节讨论了 MySQL 8.0 中可用的分区类型。这些包括以下列出的类型：

+   **RANGE 分区。** 这种分区类型根据列值落在给定范围内来将行分配到分区。参见 26.2.1 节，“RANGE 分区”。有关此类型的扩展信息，`RANGE COLUMNS`，请参见 26.2.3.1 节，“RANGE COLUMNS 分区”。

+   **LIST 分区。** 类似于`RANGE`分区，不同之处在于根据与一组离散值匹配的列来选择分区。参见 26.2.2 节，“LIST 分区”。有关此类型的扩展信息，`LIST COLUMNS`，请参见 26.2.3.2 节，“LIST COLUMNS 分区”。

+   **HASH 分区。** 使用这种分区类型，根据用户定义的表达式返回的值来选择一个分区，该表达式在要插入表中的行的列值上操作。该函数可以由 MySQL 中任何产生整数值的有效表达式组成。参见 26.2.4 节，“HASH 分区”。

    这种类型的扩展，`LINEAR HASH`，也是可用的，请参见 26.2.4.1 节，“LINEAR HASH 分区”。

+   **KEY 分区。** 这种分区类型类似于`HASH`分区，不同之处在于只提供要评估的一个或多个列，并且 MySQL 服务器提供自己的哈希函数。这些列可以包含除整数值以外的其他值，因为 MySQL 提供的哈希函数保证无论列数据类型如何，都会得到一个整数结果。这种类型的扩展，`LINEAR KEY`，也是可用的。参见 26.2.5 节，“KEY 分区”。

数据库分区的一个非常常见的用途是按日期对数据进行分隔。一些数据库系统支持显式日期分区，MySQL 在 8.0 中没有实现。但是，在 MySQL 中，基于`DATE`、`TIME`或`DATETIME`列或利用这些列的表达式创建分区方案并不困难。

当按`KEY`或`LINEAR KEY`进行分区时，您可以使用`DATE`、`TIME`或`DATETIME`列作为分区列，而无需对列值进行任何修改。例如，在 MySQL 中，以下表创建语句是完全有效的：

```sql
CREATE TABLE members (
    firstname VARCHAR(25) NOT NULL,
    lastname VARCHAR(25) NOT NULL,
    username VARCHAR(16) NOT NULL,
    email VARCHAR(35),
    joined DATE NOT NULL
)
PARTITION BY KEY(joined)
PARTITIONS 6;
```

在 MySQL 8.0 中，还可以使用`RANGE COLUMNS`和`LIST COLUMNS`分区使用`DATE`或`DATETIME`列作为分区列。

其他分区类型需要产生整数值或`NULL`的分区表达式。如果您希望通过`RANGE`、`LIST`、`HASH`或`LINEAR HASH`进行基于日期的分区，您可以简单地使用一个操作`DATE`、`TIME`或`DATETIME`列并返回这样一个值的函数，如下所示：

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

本章的以下部分中还可以找到使用日期进行分区的其他示例：

+   26.2.1 节，“RANGE 分区”

+   26.2.4 节，“HASH 分区”

+   26.2.4.1 节，“线性哈希分区”

有关基于日期的分区的更复杂示例，请参阅以下部分：

+   26.4 节，“分区修剪”

+   26.2.6 节，“子分区”

MySQL 分区针对与`TO_DAYS()`、`YEAR()`和`TO_SECONDS()`函数一起使用进行了优化。然而，你可以使用其他返回整数或`NULL`的日期和时间函数，如`WEEKDAY()`、`DAYOFYEAR()`或`MONTH()`。有关这些函数的更多信息，请参见第 14.7 节，“日期和时间函数”。

无论你使用何种分区类型，都要记住，分区在创建时总是自动按顺序编号，从`0`开始。当向分区表插入新行时，使用这些分区编号来识别正确的分区。例如，如果你的表使用了 4 个分区，这些分区被编号为`0`、`1`、`2`和`3`。对于`RANGE`和`LIST`分区类型，必须确保为每个分区编号定义了一个分区。对于`HASH`分区，用户提供的表达式必须求值为整数值。对于`KEY`分区，这个问题由 MySQL 服务器内部使用的哈希函数自动处理。

分区的名称通常遵循其他 MySQL 标识符（如表和数据库）的规则。然而，你应该注意，分区名称不区分大小写。例如，如下所示的`CREATE TABLE`语句会失败：

```sql
mysql> CREATE TABLE t2 (val INT)
 -> PARTITION BY LIST(val)(
 ->     PARTITION mypart VALUES IN (1,3,5),
 ->     PARTITION MyPart VALUES IN (2,4,6)
 -> );
ERROR 1488 (HY000): Duplicate partition name mypart
```

失败发生是因为 MySQL 在 `mypart` 和 `MyPart` 分区名称之间看不到任何区别。

当你为表指定分区数时，这必须表示为一个正的、非零的整数字面值，不能以`0.8E+01`或`6-2`这样的表达式表示，即使它求值为整数值也不行。不允许使用小数。 

在接下来的章节中，我们并不一定提供用于创建每种分区类型的语法的所有可能形式；有关此信息，请参见第 15.1.20 节，“CREATE TABLE 语句”。
