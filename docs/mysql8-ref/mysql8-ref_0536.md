> 原文：[`dev.mysql.com/doc/refman/8.0/en/table-scan-avoidance.html`](https://dev.mysql.com/doc/refman/8.0/en/table-scan-avoidance.html)

#### 10.2.1.23 避免全表扫描

`EXPLAIN`的输出在 MySQL 使用全表扫描解析查询时，在`type`列中显示`ALL`。这通常发生在以下情况下：

+   表太小，执行表扫描比使用键查找更快。对于少于 10 行且行长度较短的表，这是常见的情况。

+   `ON`或`WHERE`子句中没有可用的对索引列的限制条件。

+   比较具有常量值的索引列，并且 MySQL 已经计算（基于索引树）常量覆盖表的太大部分，表扫描会更快。参见第 10.2.1.1 节，“WHERE 子句优化”。

+   通过另一列使用基数较低的键（许多行匹配键值）。在这种情况下，MySQL 假设使用键可能需要许多键查找，并且表扫描会更快。

对于小表，表扫描通常是适当的，性能影响可以忽略不计。对于大表，尝试以下技术以避免优化器错误地选择表扫描：

+   使用`ANALYZE TABLE *`tbl_name`*`更新扫描表的键分布。参见第 15.7.3.1 节，“ANALYZE TABLE 语句”。

+   对扫描表使用`FORCE INDEX`，告诉 MySQL 表扫描比使用给定索引要昂贵：

    ```sql
    SELECT * FROM t1, t2 FORCE INDEX (*index_for_column*)
      WHERE t1.*col_name*=t2.*col_name*;
    ```

    参见第 10.9.4 节，“索引提示”。

+   使用`--max-seeks-for-key=1000`选项启动**mysqld**，或使用`SET max_seeks_for_key=1000`告诉优化器假设没有键扫描导致超过 1,000 个键查找。参见第 7.1.8 节，“服务器系统变量”。
