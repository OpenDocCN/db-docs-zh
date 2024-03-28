# B.3.5 与优化器相关的问题

> 原文：[`dev.mysql.com/doc/refman/8.0/en/optimizer-issues.html`](https://dev.mysql.com/doc/refman/8.0/en/optimizer-issues.html)

MySQL 使用基于成本的优化器来确定解析查询的最佳方式。在许多情况下，MySQL 可以计算出最佳的查询计划，但有时 MySQL 没有足够的关于手头数据的信息，必须对数据做出“有教养的”猜测。

当 MySQL 没有做出“正确”的事情时，您可以使用的工具有：

+   使用 `EXPLAIN` 语句获取有关 MySQL 如何处理查询的信息。只需在您的 `SELECT` 语句前加上关键字 `EXPLAIN` 即可使用：

    ```sql
    mysql> EXPLAIN SELECT * FROM t1, t2 WHERE t1.i = t2.i;
    ```

    `EXPLAIN` 在 Section 15.8.2, “EXPLAIN Statement” 中有更详细的讨论。

+   使用 `ANALYZE TABLE *`tbl_name`*` 来更新扫描表的键分布。参见 Section 15.7.3.1, “ANALYZE TABLE Statement”。

+   使用 `FORCE INDEX` 来告诉 MySQL 扫描表格时使用给定索引比较昂贵：

    ```sql
    SELECT * FROM t1, t2 FORCE INDEX (index_for_column)
    WHERE t1.col_name=t2.col_name;
    ```

    `USE INDEX` 和 `IGNORE INDEX` 也可能有用。参见 Section 10.9.4, “Index Hints”。

+   全局和表级 `STRAIGHT_JOIN`。参见 Section 15.2.13, “SELECT Statement”。

+   您可以调整全局或线程特定的系统变量。例如，使用 `--max-seeks-for-key=1000` 选项启动 **mysqld** 或使用 `SET max_seeks_for_key=1000` 来告诉优化器假设没有关键扫描导致超过 1,000 个关键查找。参见 Section 7.1.8, “Server System Variables”。
