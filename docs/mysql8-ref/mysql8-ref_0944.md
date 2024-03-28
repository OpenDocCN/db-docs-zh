> 原文：[`dev.mysql.com/doc/refman/8.0/en/insert-select.html`](https://dev.mysql.com/doc/refman/8.0/en/insert-select.html)

#### 15.2.7.1 INSERT ... SELECT Statement

```sql
INSERT [LOW_PRIORITY | HIGH_PRIORITY] [IGNORE]
    [INTO] *tbl_name*
    [PARTITION (*partition_name* [, *partition_name*] ...)]
    [(*col_name* [, *col_name*] ...)]
    {   SELECT ... 
      | TABLE *table_name* 
      | VALUES *row_constructor_list*
    }
    [ON DUPLICATE KEY UPDATE *assignment_list*]

*value*:
    {*expr* | DEFAULT}

*value_list*:
    *value* [, *value*] ...

*row_constructor_list*:
    ROW(*value_list*)[, ROW(*value_list*)][, ...]

*assignment*:
    *col_name* = 
          *value*
        | [*row_alias*.]*col_name*
        | [*tbl_name*.]*col_name*
        | [*row_alias*.]*col_alias*

*assignment_list*:
    *assignment* [, *assignment*] ...
```

使用 `INSERT ... SELECT`，你可以快速地从 `SELECT` 语句的结果中向表中插入许多行，该语句可以从一个或多个表中选择。例如：

```sql
INSERT INTO tbl_temp2 (fld_id)
  SELECT tbl_temp1.fld_order_id
  FROM tbl_temp1 WHERE tbl_temp1.fld_order_id > 100;
```

从 MySQL 8.0.19 开始，你可以使用 `TABLE` 语句代替 `SELECT`，如下所示：

```sql
INSERT INTO ta TABLE tb;
```

`TABLE tb` 等同于 `SELECT * FROM tb`。当需要将源表中的所有列插入目标表，并且不需要使用 WHERE 进行过滤时，这可能很有用。此外，可以使用 `ORDER BY` 按一个或多个列对 `TABLE` 中的行进行排序，并且可以使用 `LIMIT` 子句限制插入的行数。更多信息，请参见 Section 15.2.16, “TABLE Statement”。

`INSERT ... SELECT` 语句遵循以下条件，并且除非另有说明，也适用于 `INSERT ... TABLE`：

+   指定 `IGNORE` 来忽略可能导致重复键违规的行。

+   `INSERT` 语句的目标表可以出现在查询的 `SELECT` 部分的 `FROM` 子句中，或者作为由 `TABLE` 命名的表。但是，在子查询中不能插入到同一表并从同一表中选择。

    当从同一表中选择并插入时，MySQL 会创建一个内部临时表来保存 `SELECT` 中的行，然后将这些行插入目标表。但是，当 `t` 是一个 `TEMPORARY` 表时，你不能使用 `INSERT INTO t ... SELECT ... FROM t`，因为 `TEMPORARY` 表在同一语句中不能被引用两次。出于同样的原因，当 `t` 是一个临时表时，你也不能使用 `INSERT INTO t ... TABLE t`。请参见 Section 10.4.4, “Internal Temporary Table Use in MySQL”，以及 Section B.3.6.2, “TEMPORARY Table Problems”。

+   `AUTO_INCREMENT` 列的工作方式与往常一样。

+   为了确保二进制日志可以用于重新创建原始表，MySQL 不允许对 `INSERT ... SELECT` 或 `INSERT ... TABLE` 语句进行并发插入（参见 Section 10.11.3, “Concurrent Inserts”）。

+   为了避免在`SELECT`和`INSERT`引用相同表时出现模糊的列引用问题，请为`SELECT`部分中使用的每个表提供唯一别名，并在该部分中使用适当的别名限定列名。

    `TABLE`语句不支持别名。

您可以明确选择要使用`PARTITION`子句的源表或目标表（或两者）的哪些分区或子分区（或两者）。当`PARTITION`与语句的`SELECT`部分中的源表名称一起使用时，仅从其分区列表中命名的分区或子分区中选择行。当`PARTITION`与语句的`INSERT`部分的目标表名称一起使用时，必须能够将所有选定的行插入到分区列表后面命名的分区或子分区中。否则，`INSERT ... SELECT`语句将失败。有关更多信息和示例，请参阅 Section 26.5, “Partition Selection”。

`TABLE`不支持`PARTITION`子句。

对于`INSERT ... SELECT`语句，请参阅 Section 15.2.7.2, “INSERT ... ON DUPLICATE KEY UPDATE Statement”，了解在`ON DUPLICATE KEY UPDATE`子句中可以引用`SELECT`列的条件。这也适用于`INSERT ... TABLE`。

当没有`ORDER BY`子句的`SELECT`或`TABLE`语句返回行时，返回行的顺序是不确定的。这意味着，在使用复制时，不能保证这样的`SELECT`在源和副本上以相同的顺序返回行，这可能导致它们之间的不一致。为了防止这种情况发生，始终编写要使用`ORDER BY`子句进行复制的`INSERT ... SELECT`或`INSERT ... TABLE`语句，以在源和副本上产生相同的行顺序。另请参阅 Section 19.5.1.18, “Replication and LIMIT”。

由于这个问题，`INSERT ... SELECT ON DUPLICATE KEY UPDATE` 和 `INSERT IGNORE ... SELECT` 语句被标记为基于语句的复制不安全。在使用基于语句模式时，这些语句会在错误日志中产生警告，并在使用`MIXED`模式时以基于行的格式写入二进制日志。（Bug #11758262, Bug #50439）

请参阅 Section 19.2.1.1, “基于语句和基于行的复制的优缺点”。
