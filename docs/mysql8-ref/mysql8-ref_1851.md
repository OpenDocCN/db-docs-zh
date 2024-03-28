# 26.2.6 子分区化

> 原文：[`dev.mysql.com/doc/refman/8.0/en/partitioning-subpartitions.html`](https://dev.mysql.com/doc/refman/8.0/en/partitioning-subpartitions.html)

子分区化，也称为复合分区化，是对分区表中每个分区的进一步划分。考虑以下`CREATE TABLE`语句：

```sql
CREATE TABLE ts (id INT, purchased DATE)
    PARTITION BY RANGE( YEAR(purchased) )
    SUBPARTITION BY HASH( TO_DAYS(purchased) )
    SUBPARTITIONS 2 (
        PARTITION p0 VALUES LESS THAN (1990),
        PARTITION p1 VALUES LESS THAN (2000),
        PARTITION p2 VALUES LESS THAN MAXVALUE
    );
```

表`ts`有 3 个`RANGE`分区。这些分区——`p0`、`p1`和`p2`——进一步划分为 2 个子分区。实际上，整个表被划分为`3 * 2 = 6`个分区。但是，由于`PARTITION BY RANGE`子句的作用，前两个仅存储`purchased`列中值小于 1990 的记录。

可以对按`RANGE`或`LIST`分区的表进行子分区化。子分区可以使用`HASH`或`KEY`分区。这也称为复合分区化。

注意

`SUBPARTITION BY HASH`和`SUBPARTITION BY KEY`通常遵循与`PARTITION BY HASH`和`PARTITION BY KEY`相同的语法规则。一个例外是，`SUBPARTITION BY KEY`（不像`PARTITION BY KEY`）目前不支持默认列，因此必须指定用于此目的的列，即使表具有显式主键。这是一个我们正在努力解决的已知问题；有关更多信息和示例，请参见子分区的问题。

还可以使用`SUBPARTITION`子句明确定义子分区，以指定各个子分区的选项。例如，以更冗长的方式创建与前面示例中所示的相同表`ts`的方法如下：

```sql
CREATE TABLE ts (id INT, purchased DATE)
    PARTITION BY RANGE( YEAR(purchased) )
    SUBPARTITION BY HASH( TO_DAYS(purchased) ) (
        PARTITION p0 VALUES LESS THAN (1990) (
            SUBPARTITION s0,
            SUBPARTITION s1
        ),
        PARTITION p1 VALUES LESS THAN (2000) (
            SUBPARTITION s2,
            SUBPARTITION s3
        ),
        PARTITION p2 VALUES LESS THAN MAXVALUE (
            SUBPARTITION s4,
            SUBPARTITION s5
        )
    );
```

这里列出了一些需要注意的语法项：

+   每个分区必须具有相同数量的子分区。

+   如果您在分区表的任何分区上明确定义了任何子分区，必须定义它们全部。换句话说，以下语句将失败：

    ```sql
    CREATE TABLE ts (id INT, purchased DATE)
        PARTITION BY RANGE( YEAR(purchased) )
        SUBPARTITION BY HASH( TO_DAYS(purchased) ) (
            PARTITION p0 VALUES LESS THAN (1990) (
                SUBPARTITION s0,
                SUBPARTITION s1
            ),
            PARTITION p1 VALUES LESS THAN (2000),
            PARTITION p2 VALUES LESS THAN MAXVALUE (
                SUBPARTITION s2,
                SUBPARTITION s3
            )
        );
    ```

    即使使用`SUBPARTITIONS 2`，此语句仍将失败。

+   每个`SUBPARTITION`子句必须至少包括一个子分区的名称。否则，您可以为子分区设置任何所需选项，或允许其假定该选项的默认设置。

+   子分区名称必须在整个表中是唯一的。例如，以下`CREATE TABLE`语句是有效的：

    ```sql
    CREATE TABLE ts (id INT, purchased DATE)
        PARTITION BY RANGE( YEAR(purchased) )
        SUBPARTITION BY HASH( TO_DAYS(purchased) ) (
            PARTITION p0 VALUES LESS THAN (1990) (
                SUBPARTITION s0,
                SUBPARTITION s1
            ),
            PARTITION p1 VALUES LESS THAN (2000) (
                SUBPARTITION s2,
                SUBPARTITION s3
            ),
            PARTITION p2 VALUES LESS THAN MAXVALUE (
                SUBPARTITION s4,
                SUBPARTITION s5
            )
        );
    ```
