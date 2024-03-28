> 原文：[`dev.mysql.com/doc/refman/8.0/en/reset-master.html`](https://dev.mysql.com/doc/refman/8.0/en/reset-master.html)

#### 15.4.1.2 RESET MASTER Statement

```sql
RESET MASTER [TO *binary_log_file_index_number*]
```

警告

谨慎使用此语句，以确保不会丢失任何需要的二进制日志文件数据和 GTID 执行历史记录。

`RESET MASTER`需要`RELOAD`权限。

对于启用了二进制日志记录的服务器（`log_bin`为`ON`），`RESET MASTER`会删除所有现有的二进制日志文件并重置二进制日志索引文件，将服务器重置为启动二进制日志记录之前的状态。会创建一个新的空二进制日志文件，以便重新启动二进制日志记录。

对于使用 GTIDs 的服务器（`gtid_mode`为`ON`），发出`RESET MASTER`会重置 GTID 执行历史记录。`gtid_purged`系统变量的值设置为空字符串（`''`），`gtid_executed`系统变量的全局值（但不是会话值）设置为空字符串，并清除`mysql.gtid_executed`表（参见 mysql.gtid_executed Table）。如果启用了 GTID 的服务器启用了二进制日志记录，`RESET MASTER`也会像上面描述的那样重置二进制日志。请注意，即使启用了 GTID 的服务器是一个禁用了二进制日志记录的副本，`RESET MASTER`也是重置 GTID 执行历史记录的方法；`RESET REPLICA`对 GTID 执行历史记录没有影响。有关重置 GTID 执行历史记录的更多信息，请参阅重置 GTID 执行历史记录。

发出不带可选`TO`子句的`RESET MASTER`会删除索引文件中列出的所有二进制日志文件，将二进制日志索引文件重置为空，并创建一个从`1`开始的新二进制日志文件。在重置后，可以使用可选的`TO`子句从不是`1`的数字开始二进制日志文件索引。

确保您使用了合理的索引号值。如果输入了不正确的值，您可以通过发出带有或不带有`TO`子句的另一个`RESET MASTER`语句来更正。如果不更正超出范围的值，服务器将无法重新启动。

以下示例演示了`TO`子句的用法：

```sql
RESET MASTER TO 1234;

SHOW BINARY LOGS;
+-------------------+-----------+-----------+
| Log_name          | File_size | Encrypted |
+-------------------+-----------+-----------+
| source-bin.001234 |       154 | No        |
+-------------------+-----------+-----------+
```

重要提示

不带`TO`子句的`RESET MASTER`的效果与`PURGE BINARY LOGS`的效果有 2 个关键区别：

1.  `RESET MASTER`会删除索引文件中列出的*所有*二进制日志文件，只留下一个带有数字后缀`.000001`的空二进制日志文件，而编号不会被`PURGE BINARY LOGS`重置。

1.  当任何副本正在运行时，*不*应使用`RESET MASTER`。在副本正在运行时使用`RESET MASTER`的行为是未定义的（因此不受支持），而可以在副本正在运行时安全地使用`PURGE BINARY LOGS`。

另请参阅 Section 15.4.1.1, “PURGE BINARY LOGS Statement”。

在首次设置源和副本时，执行不带`TO`子句的`RESET MASTER`可能会很有用，以便您可以验证设置如下：

1.  启动源和副本，并开始复制（参见 Section 19.1.2, “Setting Up Binary Log File Position Based Replication”）。

1.  在源上执行几个测试查询。

1.  检查查询是否已被复制到副本。

1.  当复制正常运行时，在副本上依次执行`STOP REPLICA`，然后执行`RESET REPLICA`，然后验证副本上不存在来自测试查询的不需要的数据。

1.  从源中删除不需要的数据，然后执行`RESET MASTER`以清除与其关联的任何二进制日志条目和标识符。

在验证设置、重置源和副本并确保源或副本上没有任何不需要的数据或测试生成的二进制日志文件后，您可以启动副本并开始复制。
