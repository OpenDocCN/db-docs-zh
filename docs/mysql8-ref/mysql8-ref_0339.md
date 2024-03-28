> 原文：[`dev.mysql.com/doc/refman/8.0/en/making-trace-files.html`](https://dev.mysql.com/doc/refman/8.0/en/making-trace-files.html)

#### 7.9.1.2 创建跟踪文件

如果**mysqld**服务器无法启动或容易崩溃，您可以尝试创建一个跟踪文件来查找问题。

要执行此操作，您必须使用已编译调试支持的**mysqld**。您可以通过执行 `mysqld -V` 来检查这一点。如果版本号以 `-debug` 结尾，则编译时支持跟踪文件。（在 Windows 上，调试服务器的名称为**mysqld-debug**而不是**mysqld**。）

在 Unix 上以 `/tmp/mysqld.trace` 或在 Windows 上以 `\mysqld.trace` 的方式启动**mysqld**服务器：

```sql
$> mysqld --debug
```

在 Windows 上，您还应该使用`--standalone`标志，以便不将**mysqld**作为服务启动。在控制台窗口中，使用以下命令：

```sql
C:\> mysqld-debug --debug --standalone
```

之后，您可以在第二个控制台窗口中使用 `mysql.exe` 命令行工具重现问题。您可以使用**mysqladmin shutdown**停止**mysqld**服务器。

跟踪文件可能会变得**非常大**！要生成较小的跟踪文件，您可以使用类似以下的调试选项：

**mysqld --debug=d,info,error,query,general,where:O,/tmp/mysqld.trace**

这仅将最有趣的标签信息打印到跟踪文件中。

如果您提交错误报告，请仅将跟踪文件中指示出错位置的那些行添加到错误报告中。如果找不到错误位置，请打开错误报告并将整个跟踪文件上传到报告中，以便 MySQL 开发人员查看。有关说明，请参见第 1.5 节，“如何报告错误或问题”。

跟踪文件是由 Fred Fish 的 `DBUG` 软件包制作的。请参见第 7.9.4 节，“DBUG 软件包”。
