> 原文：[`dev.mysql.com/doc/refman/8.0/en/error-log-rotation.html`](https://dev.mysql.com/doc/refman/8.0/en/error-log-rotation.html)

#### 7.4.2.10 错误日志文件清空和重命名

如果使用`FLUSH ERROR LOGS`或`FLUSH LOGS`语句，或者使用**mysqladmin flush-logs**命令来清空错误日志，服务器将关闭并重新打开正在写入的任何错误日志文件。在清空之前，请手动重命名错误日志文件。清空日志后，将使用原始文件名打开一个新文件。例如，假设日志文件名为`*`host_name`*.err`，请使用以下命令重命名文件并创建一个新文件：

```sql
mv *host_name*.err *host_name*.err-old
mysqladmin flush-logs error
mv *host_name*.err-old *backup-directory*
```

在 Windows 上，请使用**rename**而不是**mv**。

如果服务器无法写入错误日志文件的位置，则清空日志操作将无法创建新的日志文件。例如，在 Linux 上，服务器可能会将错误日志写入`/var/log/mysqld.log`文件，其中`/var/log`目录由`root`所有且不可写入**mysqld**。有关处理此情况的信息，请参见第 7.4.6 节“服务器日志维护”。

如果服务器没有写入命名的错误日志文件，则在清空错误日志时不会发生错误日志文件重命名。
