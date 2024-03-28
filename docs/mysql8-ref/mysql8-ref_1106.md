> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-profile.html`](https://dev.mysql.com/doc/refman/8.0/en/show-profile.html)

#### 15.7.7.30 SHOW PROFILE Statement

```sql
SHOW PROFILE [*type* [, *type*] ... ]
    [FOR QUERY *n*]
    [LIMIT *row_count* [OFFSET *offset*]]

*type*: {
    ALL
  | BLOCK IO
  | CONTEXT SWITCHES
  | CPU
  | IPC
  | MEMORY
  | PAGE FAULTS
  | SOURCE
  | SWAPS
}
```

`SHOW PROFILE`和`SHOW PROFILES`语句显示有关在当前会话期间执行的语句的资源使用情况的分析信息。

注意

`SHOW PROFILE`和`SHOW PROFILES`语句已弃用；预计它们将在未来的 MySQL 版本中被移除。请改用性能模式；参见第 29.19.1 节，“使用性能模式进行查询分析”。

要控制分析，使用`profiling`会话变量，默认值为 0（`OFF`）。通过将`profiling`设置为 1 或`ON`来启用分析：

```sql
mysql> SET profiling = 1;
```

`SHOW PROFILES`显示发送到服务器的最近语句列表。列表的大小由`profiling_history_size`会话变量控制，默认值为 15。最大值为 100。将值设置为 0 的实际效果是禁用分析。

所有语句都会被分析，除了`SHOW PROFILE`和`SHOW PROFILES`，因此这两个语句都不会出现在分析列表中。格式错误的语句会被分析。例如，`SHOW PROFILING`是一个非法语句，如果尝试执行它，会发生语法错误，但它会出现在分析列表中。

`SHOW PROFILE`显示关于单个语句的详细信息。如果没有`FOR QUERY *`n`*`子句，则输出与最近执行的语句相关。如果包括`FOR QUERY *`n`*`，`SHOW PROFILE`显示语句*n*的信息。*n*的值对应于`SHOW PROFILES`显示的`Query_ID`值。

可以使用`LIMIT *`row_count`*`子句来限制输出为*`row_count`*行。如果给出`LIMIT`，则可以添加`OFFSET *`offset`*`以从完整行集的第*`offset`*行开始输出。

默认情况下，`SHOW PROFILE` 显示`Status`和`Duration`列。`Status`值类似于`SHOW PROCESSLIST`显示的`State`值，尽管对于某些状态值，这两个语句的解释可能存在一些细微差异（请参阅第 10.14 节，“检查服务器线程（进程）信息” Information")）。

可以指定可选的*`type`*值以显示特定的附加信息：

+   `ALL` 显示所有信息

+   `BLOCK IO` 显示块输入和输出操作的计数

+   `CONTEXT SWITCHES` 显示自愿和非自愿上下文切换的计数

+   `CPU` 显示用户和系统 CPU 使用时间

+   `IPC` 显示发送和接收消息的计数

+   `MEMORY` 目前尚未实现

+   `PAGE FAULTS` 显示主要和次要页面错误的计数

+   `SOURCE` 显示源代码中函数的名称，以及函数出现的文件的名称和行号

+   `SWAPS` 显示交换计数

会话级别启用了性能分析。当会话结束时，其性能分析信息将丢失。

```sql
mysql> SELECT @@profiling;
+-------------+
| @@profiling |
+-------------+
|           0 |
+-------------+
1 row in set (0.00 sec)

mysql> SET profiling = 1;
Query OK, 0 rows affected (0.00 sec)

mysql> DROP TABLE IF EXISTS t1;
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> CREATE TABLE T1 (id INT);
Query OK, 0 rows affected (0.01 sec)

mysql> SHOW PROFILES;
+----------+----------+--------------------------+
| Query_ID | Duration | Query                    |
+----------+----------+--------------------------+
|        0 | 0.000088 | SET PROFILING = 1        |
|        1 | 0.000136 | DROP TABLE IF EXISTS t1  |
|        2 | 0.011947 | CREATE TABLE t1 (id INT) |
+----------+----------+--------------------------+
3 rows in set (0.00 sec)

mysql> SHOW PROFILE;
+----------------------+----------+
| Status               | Duration |
+----------------------+----------+
| checking permissions | 0.000040 |
| creating table       | 0.000056 |
| After create         | 0.011363 |
| query end            | 0.000375 |
| freeing items        | 0.000089 |
| logging slow query   | 0.000019 |
| cleaning up          | 0.000005 |
+----------------------+----------+
7 rows in set (0.00 sec)

mysql> SHOW PROFILE FOR QUERY 1;
+--------------------+----------+
| Status             | Duration |
+--------------------+----------+
| query end          | 0.000107 |
| freeing items      | 0.000008 |
| logging slow query | 0.000015 |
| cleaning up        | 0.000006 |
+--------------------+----------+
4 rows in set (0.00 sec)

mysql> SHOW PROFILE CPU FOR QUERY 2;
+----------------------+----------+----------+------------+
| Status               | Duration | CPU_user | CPU_system |
+----------------------+----------+----------+------------+
| checking permissions | 0.000040 | 0.000038 |   0.000002 |
| creating table       | 0.000056 | 0.000028 |   0.000028 |
| After create         | 0.011363 | 0.000217 |   0.001571 |
| query end            | 0.000375 | 0.000013 |   0.000028 |
| freeing items        | 0.000089 | 0.000010 |   0.000014 |
| logging slow query   | 0.000019 | 0.000009 |   0.000010 |
| cleaning up          | 0.000005 | 0.000003 |   0.000002 |
+----------------------+----------+----------+------------+
7 rows in set (0.00 sec)
```

注意

在某些架构上，性能分析仅部分功能可用。对于依赖于`getrusage()`系统调用的值，在不支持该调用的系统（如 Windows）上将返回`NULL`。此外，性能分析是针对进程而不是线程的。这意味着服务器中除了您自己的线程之外的其他线程的活动可能会影响您看到的时间信息。

性能分析信息也可以从`INFORMATION_SCHEMA` `PROFILING`表中获取。请参阅第 28.3.24 节，“INFORMATION_SCHEMA PROFILING 表”。例如，以下查询是等效的：

```sql
SHOW PROFILE FOR QUERY 2;

SELECT STATE, FORMAT(DURATION, 6) AS DURATION
FROM INFORMATION_SCHEMA.PROFILING
WHERE QUERY_ID = 2 ORDER BY SEQ;
```
