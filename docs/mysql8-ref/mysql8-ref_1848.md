# 26.2.4 哈希分区

> 原文：[`dev.mysql.com/doc/refman/8.0/en/partitioning-hash.html`](https://dev.mysql.com/doc/refman/8.0/en/partitioning-hash.html)

26.2.4.1 线性哈希分区

主要使用`HASH`分区来确保数据在预定数量的分区之间均匀分布。使用范围或列表分区，您必须明确指定给定列值或一组列值应存储在哪个分区；使用哈希分区，这个决定已经为您处理，您只需指定一个基于列值的列值或表达式进行哈希处理，以及要将分区表分成的分区数。

要使用`HASH`分区对表进行分区，必须在`CREATE TABLE`语句中追加一个`PARTITION BY HASH (*`expr`*)`子句，其中*`expr`*是返回整数的表达式。这可以简单地是 MySQL 整数类型之一的列名。此外，您很可能希望在此之后跟上`PARTITIONS *`num`*`，其中*`num`*是表示要将表分成的分区数的正整数。

注意

为简单起见，接下来的示例中的表不使用任何键。您应该知道，如果表具有任何唯一键，那么用于此表的分区表达式中的每一列都必须是每个唯一键的一部分，包括主键。有关更多信息，请参见第 26.6.1 节“分区键、主键和唯一键”。

以下语句创建了一个表，该表在`store_id`列上使用哈希分区，并分为 4 个分区：

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
PARTITION BY HASH(store_id)
PARTITIONS 4;
```

如果不包括`PARTITIONS`子句，则分区数默认为`1`；在`PARTITIONS`关键字后没有跟随数字会导致语法错误。

您还可以使用返回整数的 SQL 表达式作为*`expr`*。例如，您可能希望根据雇员入职的年份进行分区。可以按照这里所示进行操作：

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
PARTITION BY HASH( YEAR(hired) )
PARTITIONS 4;
```

*`expr`*必须返回一个非常量、非随机的整数值（换句话说，它应该是变化的但确定的），并且不得包含任何禁止的构造，如第 26.6 节“分区的限制和限制”中所述。您还应该记住，每次插入、更新（或可能删除）一行时，都会评估此表达式；这意味着非常复杂的表达式可能会导致性能问题，特别是在执行影响大量行的操作（如批量插入）时。

最有效的哈希函数是一个仅作用于单个表列的函数，其值随着列值的增加或减少而一致变化，因为这允许在分区范围上进行“修剪”。也就是说，表达式与其基于的列值的变化越密切，MySQL 就能更有效地使用该表达式进行哈希分区。

例如，当`date_col`是`DATE`类型的列时，表达式`TO_DAYS(date_col)`被认为与`date_col`的值直接变化，因为对于`date_col`值的每一次变化，表达式的值都以一种一致的方式变化。与`date_col`相关的表达式`YEAR(date_col)`的变化与`date_col`不像`TO_DAYS(date_col)`那么直接，因为并非每一次`date_col`的变化都会产生与`YEAR(date_col)`等效的变化。即便如此，`YEAR(date_col)`是一个很好的哈希函数候选，因为它与`date_col`的一部分直接变化，并且没有任何可能的`date_col`变化会导致`YEAR(date_col)`的不成比例变化。

相比之下，假设你有一个名为`int_col`的列，其类型是`INT`。现在考虑表达式`POW(5-int_col,3) + 6`。这将是一个糟糕的哈希函数选择，因为`int_col`值的变化不能保证产生表达式值的成比例变化。通过给定量改变`int_col`的值可能会导致表达式值的差异很大。例如，将`int_col`从`5`更改为`6`会导致表达式值减少`-1`，但将`int_col`的值从`6`更改为`7`会导致表达式值减少`-7`。

换句话说，列值与表达式值之间的图形越接近由方程`y=*`c`*x`跟踪的直线，其中*`c`*是某个非零常数，表达式越适合用于哈希。这与表达式越非线性，数据在分区之间分布越不均匀有关。

理论上，对涉及多个列值的表达式也可以进行修剪，但确定哪些表达式适合可能会非常困难和耗时。因此，不建议特别使用涉及多个列的哈希表达式。

当使用`PARTITION BY HASH`时，存储引擎根据表达式的结果的模数确定使用哪个*`num`*分区。换句话说，对于给定的表达式*`expr`*，记录存储在分区号*`N`*中，其中`*`N`* = MOD(*`expr`*, *`num`*)`。假设表`t1`定义如下，因此有 4 个分区：

```sql
CREATE TABLE t1 (col1 INT, col2 CHAR(5), col3 DATE)
    PARTITION BY HASH( YEAR(col3) )
    PARTITIONS 4;
```

如果你向`t1`插入一个`col3`值为`'2005-09-15'`的记录，则存储它的分区如下确定：

```sql
MOD(YEAR('2005-09-01'),4)
=  MOD(2005,4)
=  1
```

MySQL 8.0 还支持一种称为线性哈希的`HASH`分区的变体，它采用更复杂的算法来确定插入到分区表中的新行的位置。请参阅 Section 26.2.4.1, “线性哈希分区”，了解此算法的描述。

用户提供的表达式在每次插入或更新记录时进行评估。根据情况，当记录被删除时也可能进行评估。
