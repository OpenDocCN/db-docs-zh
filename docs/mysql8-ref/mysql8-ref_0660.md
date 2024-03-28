# 11.2.3 标识符大小写敏感性

> 原文：[`dev.mysql.com/doc/refman/8.0/en/identifier-case-sensitivity.html`](https://dev.mysql.com/doc/refman/8.0/en/identifier-case-sensitivity.html)

在 MySQL 中，数据库对应于数据目录中的目录。数据库中的每个表对应于数据库目录中的至少一个文件（根据存储引擎可能还有更多）。触发器也对应于文件。因此，底层操作系统的大小写敏感性在数据库、表和触发器名称的大小写敏感性中起作用。这意味着在 Windows 中这些名称不区分大小写，但在大多数 Unix 变体中是区分大小写的。一个值得注意的例外是 macOS，它基于 Unix 但使用的默认文件系统类型（HFS+）不区分大小写。但是，macOS 也支持大小写敏感的 UFS 卷，就像在任何 Unix 上一样。参见第 1.6.1 节，“MySQL 对标准 SQL 的扩展”。`lower_case_table_names`系统变量还影响服务器处理标识符大小写敏感性的方式，如本节后面所述。

注意

虽然在某些平台上数据库、表和触发器名称不区分大小写，但在同一语句中不应该使用不同大小写来引用其中之一。以下语句不起作用，因为它既引用了表`my_table`又引用了`MY_TABLE`：

```sql
mysql> SELECT * FROM my_table WHERE MY_TABLE.col=1;
```

分区、子分区、列、索引、存储过程、事件和资源组名称在任何平台上都不区分大小写，列别名也是如此。

然而，日志文件组的名称是区分大小写的。这与标准 SQL 不同。

默认情况下，在 Unix 上表别名区分大小写，但在 Windows 或 macOS 上不是。以下语句在 Unix 上不起作用，因为它既引用了别名`a`又引用了`A`：

```sql
mysql> SELECT *col_name* FROM *tbl_name* AS a
       WHERE a.*col_name* = 1 OR A.*col_name* = 2;
```

然而，在 Windows 上允许相同的语句。为避免由这些差异引起的问题，最好采用一致的约定，例如始终使用小写名称创建和引用数据库和表。这种约定建议用于最大的可移植性和易用性。

MySQL 中表和数据库名称在磁盘上的存储和使用受到`lower_case_table_names`系统变量的影响。`lower_case_table_names`可以采用以下表中显示的值。此变量*不*影响触发器标识符的大小写敏感性。在 Unix 上，`lower_case_table_names`的默认值为 0。在 Windows 上，默认值为 1。在 macOS 上，默认值为 2。

只能在初始化服务器时配置`lower_case_table_names`。在服务器初始化后更改`lower_case_table_names`设置是被禁止的。

| 值 | 含义 |
| --- | --- |
| `0` | 表和数据库名称在磁盘上使用`CREATE TABLE`或`CREATE DATABASE`语句中指定的大小写形式存储。名称比较区分大小写。如果你在具有不区分大小写文件名的系统上运行 MySQL（如 Windows 或 macOS），不应将此变量设置为 0。如果你在不区分大小写的文件系统上强制将此变量设置为 0，并使用不同大小写访问`MyISAM`表名，可能会导致索引损坏。 |
| `1` | 表名在磁盘上以小写形式存储，名称比较不区分大小写。MySQL 在存储和查找时将所有表名转换为小写。这种行为也适用于数据库名称和表别名。 |
| `2` | 表和数据库名称在磁盘上使用`CREATE TABLE`或`CREATE DATABASE`语句中指定的大小写形式存储，但 MySQL 在查找时将它们转换为小写。名称比较不区分大小写。这仅适用于不区分大小写的文件系统！`InnoDB`表名和视图名以小写形式存储，就像`lower_case_table_names=1`一样。 |

如果你只在一个平台上使用 MySQL，通常不需要使用除默认值以外的`lower_case_table_names`设置。但是，如果你想在文件系统大小写敏感性不同的平台之间传输表格时可能会遇到困难。例如，在 Unix 上，你可以有两个不同的表格命名为`my_table`和`MY_TABLE`，但在 Windows 上这两个名称被视为相同。为了避免由数据库或表名大小写引起的数据传输问题，你有两个选择：

+   在所有系统上使用`lower_case_table_names=1`。这样做的主要缺点是，当你使用`SHOW TABLES`或`SHOW DATABASES`时，你看不到原始大小写的名称。

+   在 Unix 上使用`lower_case_table_names=0`，在 Windows 上使用`lower_case_table_names=2`。这将保留数据库和表名称的大小写。这样做的缺点是，您必须确保在 Windows 上始终使用正确的大小写引用您的数据库和表名称。如果您将您的语句转移到大小写敏感的 Unix 系统，如果大小写不正确，它们将无法正常工作。

    **异常**：如果您正在使用`InnoDB`表，并且正在尝试避免这些数据传输问题，您应该在所有平台上使用`lower_case_table_names=1`来强制将名称转换为小写。

如果根据二进制排序规则，对象名称的大写形式相等，则可能被视为重复。这适用于游标、条件、存储过程、函数、保存点、存储过程参数、存储程序局部变量和插件的名称。但对于列、约束、数据库、分区、使用`PREPARE`准备的语句、表、触发器、用户和用户定义变量的名称则不适用。

文件系统的大小写敏感性可能会影响`INFORMATION_SCHEMA`表中字符串列的搜索。有关更多信息，请参见第 12.8.7 节，“在 INFORMATION_SCHEMA 搜索中使用排序规则”。
