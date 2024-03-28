# 26.3 分区管理

> 原文：[`dev.mysql.com/doc/refman/8.0/en/partitioning-management.html`](https://dev.mysql.com/doc/refman/8.0/en/partitioning-management.html)

26.3.1 RANGE 和 LIST 分区管理

26.3.2 HASH 和 KEY 分区管理

26.3.3 与表交换分区和子分区

26.3.4 分区维护

26.3.5 获取分区信息

有多种使用 SQL 语句修改分区表的方法；可以通过使用分区扩展来添加、删除、重新定义、合并或拆分现有分区来修改分区表，这些操作都是通过 `ALTER TABLE` 语句完成的。还有一些方法可以获取关于分区表和分区的信息。我们将在接下来的章节中讨论这些主题。

+   有关在按 `RANGE` 或 `LIST` 分区的表中进行分区管理的信息，请参见 第 26.3.1 节，“RANGE 和 LIST 分区的管理”。

+   有关管理 `HASH` 和 `KEY` 分区的讨论，请参见 第 26.3.2 节，“HASH 和 KEY 分区的管理”。

+   请参见 第 26.3.5 节，“获取分区信息”，了解 MySQL 8.0 提供的用于获取关于分区表和分区信息的机制。

+   有关对分区执行维护操作的讨论，请参见 第 26.3.4 节，“分区维护”。

注意

所有分区表的分区必须具有相同数量的子分区；一旦表被创建，就无法更改子分区。

要更改表的分区方案，只需使用带有 *`partition_options`* 选项的 `ALTER TABLE` 语句，其语法与用于创建分区表的 `CREATE TABLE` 相同；这个选项总是以关键字 `PARTITION BY` 开头。假设以下 `CREATE TABLE` 语句用于创建一个按范围分区的表：

```sql
CREATE TABLE trb3 (id INT, name VARCHAR(50), purchased DATE)
    PARTITION BY RANGE( YEAR(purchased) ) (
        PARTITION p0 VALUES LESS THAN (1990),
        PARTITION p1 VALUES LESS THAN (1995),
        PARTITION p2 VALUES LESS THAN (2000),
        PARTITION p3 VALUES LESS THAN (2005)
    );
```

要将此表重新分区，使其按键分为两个分区，使用 `id` 列值作为键的基础，可以使用以下语句：

```sql
ALTER TABLE trb3 PARTITION BY KEY(id) PARTITIONS 2;
```

这对表的结构具有与删除表并使用 `CREATE TABLE trb3 PARTITION BY KEY(id) PARTITIONS 2;` 重新创建相同的效果。

`ALTER TABLE ... ENGINE = ...`仅更改表使用的存储引擎，并保留表的分区方案不变。该语句仅在目标存储引擎提供分区支持时成功。您可以使用`ALTER TABLE ... REMOVE PARTITIONING`来移除表的分区；参见 Section 15.1.9, “ALTER TABLE Statement”。

重要提示

在给定的`ALTER TABLE`语句中只能使用单个`PARTITION BY`、`ADD PARTITION`、`DROP PARTITION`、`REORGANIZE PARTITION`或`COALESCE PARTITION`子句。如果您（例如）希望删除一个分区并重新组织表的其余分区，您必须在两个单独的`ALTER TABLE`语句中执行此操作（一个使用`DROP PARTITION`，然后第二个使用`REORGANIZE PARTITION`）。

你可以使用`ALTER TABLE ... TRUNCATE PARTITION`从一个或多个选定的分区中删除所有行。
