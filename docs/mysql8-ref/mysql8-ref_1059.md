> 原文：[`dev.mysql.com/doc/refman/8.0/en/analyze-table.html`](https://dev.mysql.com/doc/refman/8.0/en/analyze-table.html)

#### 15.7.3.1 ANALYZE TABLE Statement

```sql
ANALYZE [NO_WRITE_TO_BINLOG | LOCAL]
    TABLE *tbl_name* [, *tbl_name*] ...

ANALYZE [NO_WRITE_TO_BINLOG | LOCAL]
    TABLE *tbl_name*
    UPDATE HISTOGRAM ON *col_name* [, *col_name*] ...
        [WITH *N* BUCKETS]

ANALYZE [NO_WRITE_TO_BINLOG | LOCAL] 
    TABLE *tbl_name*
    UPDATE HISTOGRAM ON *col_name* [USING DATA '*json_data*']

ANALYZE [NO_WRITE_TO_BINLOG | LOCAL]
    TABLE *tbl_name*
    DROP HISTOGRAM ON *col_name* [, *col_name*] ...
```

`ANALYZE TABLE`生成表统计信息：

+   `ANALYZE TABLE` 在没有`HISTOGRAM`子句的情况下执行键分布分析，并为指定的表或表存储分布。对于`MyISAM`表，进行键分布分析的`ANALYZE TABLE`等同于使用**myisamchk --analyze**。

+   带有`UPDATE HISTOGRAM`子句的`ANALYZE TABLE`为指定表列生成直方图统计信息，并将其存储在数据字典中。此语法仅允许一个表名。MySQL 8.0.31 及更高版本还支持将单个列的直方图设置为用户定义的 JSON 值。

+   带有`DROP HISTOGRAM`子句的`ANALYZE TABLE`从数据字典中删除指定表列的直方图统计信息。此语法仅允许一个表名。

此语句需要表的`SELECT`和`INSERT`权限。

`ANALYZE TABLE`适用于`InnoDB`、`NDB`和`MyISAM`表。它不适用于视图。

如果启用了`innodb_read_only`系统变量，则`ANALYZE TABLE`可能会失败，因为它无法更新使用`InnoDB`的数据字典中的统计表，用于更新键分布的`ANALYZE TABLE`操作，即使操作更新表本身（例如，如果是`MyISAM`表），也可能会发生失败。要获取更新后的分布统计信息，请设置`information_schema_stats_expiry=0`。

支持对分区表进行`ANALYZE TABLE`操作，您可以使用`ALTER TABLE ... ANALYZE PARTITION`来分析一个或多个分区；有关更多信息，请参见第 15.1.9 节，“ALTER TABLE Statement”和第 26.3.4 节，“Partitions 的维护”。

在分析过程中，对于`InnoDB`和`MyISAM`，表将被读锁定。

默认情况下，服务器将`ANALYZE TABLE`语句写入二进制日志，以便它们复制到副本。要禁止记录日志，请指定可选的`NO_WRITE_TO_BINLOG`关键字或其别名`LOCAL`。

以前，`ANALYZE TABLE`需要一个刷新锁。这意味着，当调用`ANALYZE TABLE`时，如果仍有长时间运行的语句或事务在使用表，则任何后续语句和事务都必须等待这些操作完成，然后才能释放刷新锁。在 MySQL 8.0.24（及更高版本）中解决了这个问题，`ANALYZE TABLE`不再导致后续操作等待。

+   ANALYZE TABLE 输出

+   键分布分析

+   直方图统计分析

+   其他考虑

##### ANALYZE TABLE 输出

`ANALYZE TABLE` 返回一个包含以下表中所示列的结果集。

| 列 | 值 |
| --- | --- |
| `Table` | 表名 |
| `Op` | `analyze` 或 `histogram` |
| `Msg_type` | `status`, `error`, `info`, `note`, 或 `warning` |
| `Msg_text` | 一个信息性消息 |

##### 键分布分析

`ANALYZE TABLE`没有`HISTOGRAM`子句时执行键分布分析并存储表或表的分布。任何现有的直方图统计数据保持不变。

如果表自上次键分布分析以来未发生更改，则不会再次分析该表。

MySQL 使用存储的键分布来决定除常数外其他内容的连接应该以什么顺序连接表。此外，在决定查询中特定表使用哪些索引时，可以使用键分布。

要检查存储的键分布基数，使用`SHOW INDEX`语句或`INFORMATION_SCHEMA` `STATISTICS`表。参见第 15.7.7.22 节，“SHOW INDEX Statement”和第 28.3.34 节，“The INFORMATION_SCHEMA STATISTICS Table”。

对于`InnoDB`表，`ANALYZE TABLE`通过在每个索引树上执行随机潜水并相应地更新索引基数估计来确定索引基数。由于这些只是估计值，多次运行`ANALYZE TABLE`可能会产生不同的数字。这使得`ANALYZE TABLE`在`InnoDB`表上运行速度快，但不是 100%准确，因为它没有考虑所有行。

通过启用`innodb_stats_persistent`，可以使`ANALYZE TABLE`收集的统计信息更加精确和稳定，如第 17.8.10.1 节，“配置持久性优化器统计参数”中所解释的那样。在启用`innodb_stats_persistent`时，重要的是在索引列数据发生重大更改后运行`ANALYZE TABLE`，因为统计信息不会定期重新计算（例如在服务器重新启动后）。

如果启用了`innodb_stats_persistent`，可以通过修改`innodb_stats_persistent_sample_pages`系统变量来更改随机潜水次数。如果禁用了`innodb_stats_persistent`，则改为修改`innodb_stats_transient_sample_pages`。

有关`InnoDB`中键分布分析的更多信息，请参见第 17.8.10.1 节，“配置持久性优化器统计参数”和第 17.8.10.3 节，“估算 InnoDB 表的 ANALYZE TABLE 复杂度”。

MySQL 在连接优化中使用索引基数估计。如果连接没有以正确的方式优化，请尝试运行`ANALYZE TABLE`。在极少数情况下，`ANALYZE TABLE`无法为您的特定表生成足够好的值，您可以在查询中使用`FORCE INDEX`强制使用特定索引，或者设置`max_seeks_for_key`系统变量以确保 MySQL 优先选择索引查找而不是表扫描。参见第 B.3.5 节，“与优化器相关的问题”。

##### 直方图统计分析

带有`HISTOGRAM`子句的`ANALYZE TABLE`启用了对表列值的直方图统计管理。有关直方图统计信息，请参见第 10.9.6 节，“优化器统计信息”。

可用的直方图操作如下：

+   带有`UPDATE HISTOGRAM`子句的`ANALYZE TABLE`为命名表列生成直方图统计信息，并将其存储在数据字典中。此语法仅允许一个表名。

    可选的`WITH *`N`* BUCKETS`子句指定直方图的桶数。*`N`*的值必须是 1 到 1024 之间的整数。如果省略此子句，则桶数为 100。

+   带有`DROP HISTOGRAM`子句的`ANALYZE TABLE`从数据字典中删除了命名表列的直方图统计信息。此语法仅允许一个表名。

存储的直方图管理语句仅影响指定的列。考虑以下语句：

```sql
ANALYZE TABLE t UPDATE HISTOGRAM ON c1, c2, c3 WITH 10 BUCKETS;
ANALYZE TABLE t UPDATE HISTOGRAM ON c1, c3 WITH 10 BUCKETS;
ANALYZE TABLE t DROP HISTOGRAM ON c2;
```

第一条语句更新了`c1`、`c2`和`c3`列的直方图，替换了这些列的任何现有直方图。第二条语句更新了`c1`和`c3`列的直方图，而`c2`列的直方图保持不变。第三条语句移除了`c2`列的直方图，而`c1`和`c3`列的直方图保持不变。

在对用户数据进行抽样以构建直方图时，并非所有值都会被读取；这可能导致遗漏一些被认为重要的值。在这种情况下，修改直方图或根据自己的标准明确设置自己的直方图可能是有用的，例如完整数据集。MySQL 8.0.31 添加了对`ANALYZE TABLE *`tbl_name`* UPDATE HISTOGRAM ON *`col_name`* USING DATA '*`json_data`*'`的支持，用于使用与显示信息模式`COLUMN_STATISTICS`表中`HISTOGRAM`列值相同的 JSON 格式提供的数据更新直方图表的列。在使用 JSON 数据更新直方图时，只能修改一个列。

我们可以通过首先在表`t`的列`c1`上生成直方图来说明`USING DATA`的用法，就像这样：

```sql
mysql> ANALYZE TABLE t UPDATE HISTOGRAM ON c1;
+--------+-----------+----------+-----------------------------------------------+
| Table  | Op        | Msg_type | Msg_text                                      |
+--------+-----------+----------+-----------------------------------------------+
| mydb.t | histogram | status   | Histogram statistics created for column 'c1'. |
+--------+-----------+----------+-----------------------------------------------+
```

我们可以在`COLUMN_STATISTICS`表中看到生成的直方图：

```sql
mysql> TABLE information_schema.column_statistics\G
*************************** 1\. row ***************************
SCHEMA_NAME: mydb
 TABLE_NAME: t
COLUMN_NAME: c1
  HISTOGRAM: {"buckets": [[206, 0.0625], [456, 0.125], [608, 0.1875]],
"data-type": "int", "null-values": 0.0, "collation-id": 8, "last-updated":
"2022-10-11 16:13:14.563319", "sampling-rate": 1.0, "histogram-type":
"singleton", "number-of-buckets-specified": 100}
```

现在我们删除了直方图，当我们检查`COLUMN_STATISTICS`时，它现在是空的：

```sql
mysql> ANALYZE TABLE t DROP HISTOGRAM ON c1;
+--------+-----------+----------+-----------------------------------------------+
| Table  | Op        | Msg_type | Msg_text                                      |
+--------+-----------+----------+-----------------------------------------------+
| mydb.t | histogram | status   | Histogram statistics removed for column 'c1'. |
+--------+-----------+----------+-----------------------------------------------+

mysql> TABLE information_schema.column_statistics\G
Empty set (0.00 sec)
```

我们可以通过将先前从`COLUMN_STATISTICS`表的`HISTOGRAM`列中获取的 JSON 表示插入来恢复已删除的直方图，当我们再次查询该表时，我们可以看到直方图已恢复到先前的状态：

```sql
mysql> ANALYZE TABLE t UPDATE HISTOGRAM ON c1 
 ->     USING DATA '{"buckets": [[206, 0.0625], [456, 0.125], [608, 0.1875]],
    ->               "data-type": "int", "null-values": 0.0, "collation-id":
    ->               8, "last-updated": "2022-10-11 16:13:14.563319",
    ->               "sampling-rate": 1.0, "histogram-type": "singleton",
    ->               "number-of-buckets-specified": 100}';   
+--------+-----------+----------+-----------------------------------------------+
| Table  | Op        | Msg_type | Msg_text                                      |
+--------+-----------+----------+-----------------------------------------------+
| mydb.t | histogram | status   | Histogram statistics created for column 'c1'. |
+--------+-----------+----------+-----------------------------------------------+

mysql> TABLE information_schema.column_statistics\G
*************************** 1\. row ***************************
SCHEMA_NAME: mydb
 TABLE_NAME: t
COLUMN_NAME: c1
  HISTOGRAM: {"buckets": [[206, 0.0625], [456, 0.125], [608, 0.1875]],
"data-type": "int", "null-values": 0.0, "collation-id": 8, "last-updated":
"2022-10-11 16:13:14.563319", "sampling-rate": 1.0, "histogram-type":
"singleton", "number-of-buckets-specified": 100}
```

不支持为加密表（以避免在统计数据中暴露数据）或`TEMPORARY`表生成直方图。

直方图生成适用于除几何类型（空间数据）和`JSON`之外的所有数据类型的列。

可以为存储和虚拟生成列生成直方图。

无法为由单列唯一索引覆盖的列生成直方图。

直方图管理语句尝试尽可能执行请求的操作，并对其余部分报告诊断消息。例如，如果`UPDATE HISTOGRAM`语句命名了多个列，但其中一些列不存在或具有不受支持的数据类型，则会为其他列生成直方图，并为无效列生成消息。

直方图受以下 DDL 语句影响：

+   `DROP TABLE`会移除已删除表中的列的直方图。

+   `DROP DATABASE`会移除已删除数据库中任何表的直方图，因为该语句会删除数据库中的所有表。

+   `RENAME TABLE`不会移除直方图。相反，它会将重命名后的表的直方图重命名为与新表名相关联。

+   `ALTER TABLE`语句删除或修改列时会删除该列的直方图。

+   `ALTER TABLE ... CONVERT TO CHARACTER SET`会移除字符列的直方图，因为它们受字符集更改的影响。非字符列的直方图不受影响。

`histogram_generation_max_mem_size`系统变量控制用于直方图生成的最大内存量。全局和会话值可以在运行时设置。

更改全局`histogram_generation_max_mem_size`值需要具有足够权限设置全局系统变量的权限。更改会话`histogram_generation_max_mem_size`值需要具有足够权限设置受限会话系统变量的权限。参见 Section 7.1.9.1, “System Variable Privileges”。

如果用于直方图生成的估计数据量超过由`histogram_generation_max_mem_size`定义的限制，MySQL 会对数据进行抽样而不是全部读入内存。抽样均匀分布在整个表上。MySQL 使用`SYSTEM`抽样，这是一种基于页面级别的抽样方法。

可以查询信息模式`COLUMN_STATISTICS`表中`HISTOGRAM`列中的`sampling-rate`值，以确定用于创建直方图的数据分数。`sampling-rate`是一个介于 0.0 和 1.0 之间的数字。值为 1 表示所有数据都被读取（没有抽样）。

以下示例演示了抽样。为了确保数据量超过`histogram_generation_max_mem_size`限制，以便进行示例，先将限制设置为较低值（2000000 字节），然后为`employees`表的`birth_date`列生成直方图统计信息。

```sql
mysql> SET histogram_generation_max_mem_size = 2000000;

mysql> USE employees;

mysql> ANALYZE TABLE employees UPDATE HISTOGRAM ON birth_date WITH 16 BUCKETS\G
*************************** 1\. row ***************************
   Table: employees.employees
      Op: histogram
Msg_type: status
Msg_text: Histogram statistics created for column 'birth_date'. 
mysql> SELECT HISTOGRAM->>'$."sampling-rate"'
       FROM INFORMATION_SCHEMA.COLUMN_STATISTICS
       WHERE TABLE_NAME = "employees"
       AND COLUMN_NAME = "birth_date";
+---------------------------------+
| HISTOGRAM->>'$."sampling-rate"' |
+---------------------------------+
| 0.0491431208869665              |
+---------------------------------+
```

`sampling-rate`值为 0.0491431208869665 表示大约有 4.9%的`birth_date`列数据被读入内存以生成直方图统计信息。

截至 MySQL 8.0.19，`InnoDB`存储引擎为存储在`InnoDB`表中的数据提供了自己的抽样实现。当存储引擎不提供自己的抽样实现时，MySQL 使用的默认抽样实现需要进行全表扫描，对于大表来说代价高昂。`InnoDB`抽样实现通过避免全表扫描来提高抽样性能。

`sampled_pages_read`和`sampled_pages_skipped``INNODB_METRICS`计数器可用于监视`InnoDB`数据页的采样。（有关一般`INNODB_METRICS`计数器使用信息，请参见 Section 28.4.21, “The INFORMATION_SCHEMA INNODB_METRICS Table”。）

以下示例演示了采样计数器的使用，需要在生成直方图统计信息之前启用计数器。

```sql
mysql> SET GLOBAL innodb_monitor_enable = 'sampled%';

mysql> USE employees;

mysql> ANALYZE TABLE employees UPDATE HISTOGRAM ON birth_date WITH 16 BUCKETS\G
*************************** 1\. row ***************************
   Table: employees.employees
      Op: histogram
Msg_type: status
Msg_text: Histogram statistics created for column 'birth_date'. 
mysql> USE INFORMATION_SCHEMA;

mysql> SELECT NAME, COUNT FROM INNODB_METRICS WHERE NAME LIKE 'sampled%'\G
*************************** 1\. row ***************************
 NAME: sampled_pages_read
COUNT: 43
*************************** 2\. row ***************************
 NAME: sampled_pages_skipped
COUNT: 843
```

这个公式基于采样计数器数据近似采样率：

```sql
sampling rate = sampled_page_read/(sampled_pages_read + sampled_pages_skipped)
```

基于采样计数器数据的采样率大致等同于信息模式`COLUMN_STATISTICS`表中`HISTOGRAM`列中的`sampling-rate`值。

有关生成直方图时执行的内存分配的信息，请监视性能模式`memory/sql/histograms`工具。参见 Section 29.12.20.10, “Memory Summary Tables”。

##### 其他考虑因素

`ANALYZE TABLE`从信息模式`INNODB_TABLESTATS`表中清除表统计信息，并将`STATS_INITIALIZED`列设置为`Uninitialized`。统计信息在下次访问表时再次收集。
