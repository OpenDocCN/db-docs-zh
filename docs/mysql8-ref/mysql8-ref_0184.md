> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-batch-commands.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-batch-commands.html)

#### 6.5.1.5 从文本文件执行 SQL 语句

**mysql**客户端通常是以交互方式使用的，就像这样：

```sql
mysql *db_name*
```

但是，您也可以将 SQL 语句放在一个文件中，然后告诉**mysql**从该文件中读取输入。要这样做，请创建一个包含您希望执行的语句的文本文件 *`text_file`*。然后按照这里所示调用**mysql**：

```sql
mysql *db_name* < *text_file*
```

如果您在文件中将 `USE *`db_name`*` 语句作为第一条语句，那么在命令行上指定数据库名称是不必要的：

```sql
mysql < text_file
```

如果您已经运行**mysql**，您可以使用 `source` 命令或 `\.` 命令执行一个 SQL 脚本文件：

```sql
mysql> source *file_name*
mysql> \. *file_name*
```

有时，您可能希望脚本向用户显示进度信息。为此，您可以插入如下语句：

```sql
SELECT '<info_to_display>' AS ' ';
```

所示语句输出 `<info_to_display>`。

您还可以使用 `--verbose` 选项调用**mysql**，这会导致在产生结果之前显示每个语句。

**mysql**会忽略输入文件开头的 Unicode 字节顺序标记（BOM）字符。以前，它会读取它们并将它们发送到服务器，导致语法错误。BOM 的存在不会导致**mysql**更改其默认字符集。要这样做，请使用诸如 `--default-character-set=utf8mb4` 的选项调用**mysql**。

有关批处理模式的更多信息，请参见第 5.5 节，“在批处理模式下使用 mysql”。
