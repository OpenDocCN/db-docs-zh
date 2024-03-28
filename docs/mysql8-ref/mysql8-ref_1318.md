# 18.7.2 MERGE 表问题

> 原文：[`dev.mysql.com/doc/refman/8.0/en/merge-table-problems.html`](https://dev.mysql.com/doc/refman/8.0/en/merge-table-problems.html)

以下是 `MERGE` 表的已知问题：

+   在 MySQL Server 版本 5.1.23 之前的版本中，可以创建具有非临时子 MyISAM 表的临时合并表。

    从版本 5.1.23 开始，`MERGE` 子表通过父表进行锁定。如果父表是临时表，则不会被锁定，因此子表也不会被锁定。同时使用 MyISAM 表会导致数据损坏。

+   如果使用 `ALTER TABLE` 将 `MERGE` 表更改为另一个存储引擎，那么与基础表的映射将丢失。相反，来自基础 `MyISAM` 表的行将被复制到更改后的表中，然后使用指定的存储引擎。

+   `MERGE` 表的 `INSERT_METHOD` 表选项指示要用于向 `MERGE` 表插入的基础 `MyISAM` 表。然而，在至少有一行直接插入到 `MyISAM` 表之前，对该 `MyISAM` 表使用 `AUTO_INCREMENT` 表选项对向 `MERGE` 表的插入没有影响。

+   `MERGE` 表无法在整个表上维护唯一性约束。当执行 `INSERT` 时，数据会进入第一个或最后一个 `MyISAM` 表（由 `INSERT_METHOD` 选项确定）。MySQL 确保唯一键值在该 `MyISAM` 表内保持唯一，但不跨所有基础表。

+   因为 `MERGE` 引擎无法强制执行基础表集合上的唯一性，`REPLACE` 无法按预期工作。两个关键事实是：

    +   `REPLACE` 只能检测即将写入的基础表中的唯一键冲突（由 `INSERT_METHOD` 选项确定）。这与 `MERGE` 表本身中的冲突不同。

    +   如果 `REPLACE` 检测到唯一键冲突，它只会更改正在写入的基础表中的相应行；也就是说，由 `INSERT_METHOD` 选项确定的第一个或最后一个表。

    对于 `INSERT ... ON DUPLICATE KEY UPDATE`，类似的考虑也适用。

+   `MERGE` 表不支持分区。也就是说，无法对 `MERGE` 表进行分区，`MERGE` 表的任何基础 `MyISAM` 表也无法进行分区。

+   不应在任何映射到打开 `MERGE` 表中的表上使用 `ANALYZE TABLE`、`REPAIR TABLE`、`OPTIMIZE TABLE`、`ALTER TABLE`、`DROP TABLE`、不带 `WHERE` 子句的 `DELETE` 或 `TRUNCATE TABLE`。如果这样做，`MERGE` 表可能仍然引用原始表并产生意外结果。为了解决这个问题，在执行任何命名操作之前，确保没有任何 `MERGE` 表保持打开状态，通过发出一个 `FLUSH TABLES` 语句。

    意外结果包括 `MERGE` 表上的操作可能报告表损坏的可能性。如果在底层 `MyISAM` 表的命名操作之后发生这种情况，则损坏消息是虚假的。为了解决这个问题，在修改 `MyISAM` 表后，发出一个 `FLUSH TABLES` 语句。

+   在 Windows 上，对正在被 `MERGE` 表使用的表执行 `DROP TABLE` 操作是无效的，因为 MySQL 的 `MERGE` 存储引擎的表映射对于上层是隐藏的。Windows 不允许删除打开的文件，因此在删除表之前，你必须先刷新所有的 `MERGE` 表（使用 `FLUSH TABLES`）或者删除 `MERGE` 表。

+   当访问表时（例如，作为 `SELECT` 或 `INSERT` 语句的一部分），`MyISAM` 表和 `MERGE` 表的定义会被检查。这些检查通过比较列顺序、类型、大小和相关索引来确保表的定义和父 `MERGE` 表的定义匹配。如果表之间存在差异，将返回错误并导致语句失败。由于这些检查发生在表被打开时，对单个表的定义进行任何更改，包括列更改、列排序和引擎更改都会导致语句失败。

+   `MERGE` 表及其基础表中索引的顺序应该相同。如果使用 `ALTER TABLE` 在用于 `MERGE` 表的表上添加一个 `UNIQUE` 索引，然后再在 `MERGE` 表上添加一个非唯一索引，如果基础表中已经有一个非唯一索引，那么这些表的索引顺序就会不同。 (这是因为 `ALTER TABLE` 将 `UNIQUE` 索引放在非唯一索引之前，以便快速检测重复键。) 因此，对具有这些索引的表的查询可能会返回意外结果。

+   如果遇到类似于 ERROR 1017 (HY000): Can't find file: '*`tbl_name`*.MRG' (errno: 2) 的错误消息，通常表示一些基础表没有使用 `MyISAM` 存储引擎。确认所有这些表都是 `MyISAM`。

+   `MERGE` 表中的最大行数为 2⁶⁴ (~1.844E+19; 与 `MyISAM` 表相同)。无法将多个 `MyISAM` 表合并为一个行数超过此数量的 `MERGE` 表。

+   目前已知，使用具有不同行格式的基础 `MyISAM` 表与父 `MERGE` 表会失败。请参见 Bug #32364。

+   当 `LOCK TABLES` 生效时，您无法更改非临时 `MERGE` 表的联合列表。以下操作*不*起作用：

    ```sql
    CREATE TABLE m1 ... ENGINE=MRG_MYISAM ...;
    LOCK TABLES t1 WRITE, t2 WRITE, m1 WRITE;
    ALTER TABLE m1 ... UNION=(t1,t2) ...;
    ```

    但是，您可以使用临时 `MERGE` 表来实现这一点。

+   无法使用 `CREATE ... SELECT` 创建临时 `MERGE` 表或非临时 `MERGE` 表。例如：

    ```sql
    CREATE TABLE m1 ... ENGINE=MRG_MYISAM ... SELECT ...;
    ```

    尝试这样做会导致错误：*`tbl_name`* 不是 `BASE TABLE`。

+   在某些情况下，`MERGE` 表和基础表之间的 `PACK_KEYS` 表选项值不同会导致意外结果，如果基础表包含 `CHAR` 或 `BINARY` 列。作为解决方法，使用 `ALTER TABLE` 确保所有涉及的表具有相同的 `PACK_KEYS` 值。 (Bug #50646)
