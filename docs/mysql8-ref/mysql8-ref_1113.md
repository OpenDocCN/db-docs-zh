> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-status.html`](https://dev.mysql.com/doc/refman/8.0/en/show-status.html)

#### 15.7.7.37 显示状态语句

```sql
SHOW [GLOBAL | SESSION] STATUS
    [LIKE '*pattern*' | WHERE *expr*]
```

`显示状态`提供服务器状态信息（参见第 7.1.10 节，“服务器状态变量”）。此语句不需要任何特权，只需要连接到服务器的能力。

状态变量信息也可以从以下来源获得：

+   性能模式表。参见第 29.12.15 节，“性能模式状态变量表”。

+   **mysqladmin extended-status**命令。参见第 6.5.2 节，“mysqladmin — 一个 MySQL 服务器管理程序”。

对于`显示状态`，如果存在`LIKE`子句，则指示要匹配的变量名称。可以给出`WHERE`子句以使用更一般的条件选择行，如第 28.8 节，“SHOW 语句的扩展”中所讨论的。

`显示状态`接受可选的`GLOBAL`或`SESSION`变量范围修饰符：

+   使用`GLOBAL`修饰符，该语句显示全局状态值。全局状态变量可以表示服务器本身某个方面的状态（例如，`Aborted_connects`），或者 MySQL 所有连接的聚合状态（例如，`Bytes_received`和`Bytes_sent`）。如果变量没有全局值，则显示会话值。

+   使用`SESSION`修饰符，该语句显示当前连接的状态变量值。如果变量没有会话值，则显示全局值。`LOCAL`是`SESSION`的同义词。

+   如果没有修饰符，则默认为`SESSION`。

每个状态变量的范围在第 7.1.10 节，“服务器状态变量”中列出。

每次调用`显示状态`语句都会使用内部临时表并增加全局`Created_tmp_tables`值。

此处显示了部分输出。名称和值的列表可能与您的服务器不同。每个变量的含义在第 7.1.10 节，“服务器状态变量”中给出。

```sql
mysql> SHOW STATUS;
+--------------------------+------------+
| Variable_name            | Value      |
+--------------------------+------------+
| Aborted_clients          | 0          |
| Aborted_connects         | 0          |
| Bytes_received           | 155372598  |
| Bytes_sent               | 1176560426 |
| Connections              | 30023      |
| Created_tmp_disk_tables  | 0          |
| Created_tmp_tables       | 8340       |
| Created_tmp_files        | 60         |
...
| Open_tables              | 1          |
| Open_files               | 2          |
| Open_streams             | 0          |
| Opened_tables            | 44600      |
| Questions                | 2026873    |
...
| Table_locks_immediate    | 1920382    |
| Table_locks_waited       | 0          |
| Threads_cached           | 0          |
| Threads_created          | 30022      |
| Threads_connected        | 1          |
| Threads_running          | 1          |
| Uptime                   | 80380      |
+--------------------------+------------+
```

使用`LIKE`子句，该语句仅显示那些名称与模式匹配的变量的行：

```sql
mysql> SHOW STATUS LIKE 'Key%';
+--------------------+----------+
| Variable_name      | Value    |
+--------------------+----------+
| Key_blocks_used    | 14955    |
| Key_read_requests  | 96854827 |
| Key_reads          | 162040   |
| Key_write_requests | 7589728  |
| Key_writes         | 3813196  |
+--------------------+----------+
```
