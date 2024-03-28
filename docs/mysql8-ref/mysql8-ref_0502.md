# 9.5.1 使用二进制日志进行时间点恢复

> 原文：[`dev.mysql.com/doc/refman/8.0/en/point-in-time-recovery-binlog.html`](https://dev.mysql.com/doc/refman/8.0/en/point-in-time-recovery-binlog.html)

本节解释了使用二进制日志执行时间点恢复的一般思路。下一节，第 9.5.2 节，“使用事件位置进行时间点恢复”，通过示例详细解释了操作。

注意

本节和下一节中的许多示例使用**mysql**客户端来处理**mysqlbinlog**生成的二进制日志输出。如果您的二进制日志包含`\0`（空）字符，则除非使用`--binary-mode`选项调用，否则**mysql**无法解析该输出。

时间点恢复的信息来源是在完全备份操作之后生成的一组二进制日志文件。因此，为了允许服务器恢复到某个时间点，必须在其上启用二进制日志记录，这是 MySQL 8.0 的默认设置（参见 第 7.4.4 节，“二进制日志”）。

要从二进制日志中恢复数据，必须知道当前二进制日志文件的名称和位置。默认情况下，服务器在数据目录中创建二进制日志文件，但可以使用`--log-bin`选项指定路径名以将文件放在不同位置。要查看所有二进制日志文件的列表，请使用以下语句：

```sql
mysql> SHOW BINARY LOGS;
```

要确定当前二进制日志文件的名称，请执行以下语句：

```sql
mysql> SHOW MASTER STATUS;
```

**mysqlbinlog** 实用程序将二进制日志文件中的事件从二进制格式转换为文本，以便查看或应用。**mysqlbinlog** 有选项可根据事件时间或日志中事件位置选择二进制日志的部分。请参阅 第 6.6.9 节，“mysqlbinlog — 用于处理二进制日志文件的实用程序”。

从二进制日志应用事件会导致它们所代表的数据修改重新执行。这使得可以恢复给定时间段的数据更改。要从二进制日志应用事件，请使用**mysql**客户端处理**mysqlbinlog**输出：

```sql
$> mysqlbinlog *binlog_files* | mysql -u root -p
```

如果二进制日志文件已加密，从 MySQL 8.0.14 开始可以执行此操作，**mysqlbinlog** 无法像上面的示例那样直接读取它们，但可以使用 `--read-from-remote-server` (`-R`) 选项从服务器读取它们。例如：

```sql
$> mysqlbinlog --read-from-remote-server --host=*host_name* --port=3306  --user=root --password --ssl-mode=required  *binlog_files* | mysql -u root -p
```

在这里，选项 `--ssl-mode=required` 被用来确保从二进制日志文件中传输的数据在传输过程中受到保护，因为它以未加密的格式发送给 **mysqlbinlog**。

重要

`VERIFY_CA` 和 `VERIFY_IDENTITY` 比 `REQUIRED` 更好的选择作为 SSL 模式，因为它们有助于防止中间人攻击。要实施这些设置之一，您必须首先确保服务器的 CA 证书可靠地提供给所有在您的环境中使用它的客户端，否则将导致可用性问题。请参阅加密连接的命令选项。

查看日志内容在需要确定事件时间或位置以选择执行事件之前的部分日志内容时很有用。要查看日志中的事件，请将 **mysqlbinlog** 输出发送到分页程序：

```sql
$> mysqlbinlog *binlog_files* | more
```

或者，将输出保存在文件中，并在文本编辑器中查看文件：

```sql
$> mysqlbinlog *binlog_files* > tmpfile
$> ... *edit tmpfile* ...
```

编辑文件后，应用内容如下：

```sql
$> mysql -u root -p < tmpfile
```

如果您有多个要应用于 MySQL 服务器的二进制日志，则可以使用单个连接来应用要处理的所有二进制日志文件的内容。以下是一种方法：

```sql
$> mysqlbinlog binlog.000001 binlog.000002 | mysql -u root -p
```

另一种方法是将整个日志写入单个文件，然后处理该文件：

```sql
$> mysqlbinlog binlog.000001 >  /tmp/statements.sql
$> mysqlbinlog binlog.000002 >> /tmp/statements.sql
$> mysql -u root -p -e "source /tmp/statements.sql"
```
