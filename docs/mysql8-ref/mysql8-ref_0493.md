# 9.4.3 使用 mysqldump 以分隔文本格式转储数据

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysqldump-delimited-text.html`](https://dev.mysql.com/doc/refman/8.0/en/mysqldump-delimited-text.html)

本节描述了如何使用**mysqldump**创建分隔文本转储文件。有关重新加载此类转储文件的信息，请参见 Section 9.4.4, “Reloading Delimited-Text Format Backups”。

如果使用`--tab=*`dir_name`*`选项调用**mysqldump**，它将使用*`dir_name`*作为输出目录，并在该目录中单独转储表，每个表使用两个文件。表名是这些文件的基本名称。对于名为`t1`的表，文件名为`t1.sql`和`t1.txt`。`.sql`文件包含表的`CREATE TABLE`语句。`.txt`文件包含表数据，每行一个表行。

以下命令将`db1`数据库的内容转储到`/tmp`数据库中：

```sql
$> mysqldump --tab=/tmp db1
```

包含表数据的`.txt`文件由服务器编写，因此它们由用于运行服务器的系统帐户拥有。服务器使用`SELECT ... INTO OUTFILE`来写入文件，因此您必须具有`FILE`权限才能执行此操作，如果给定的`.txt`文件已经存在，则会发生错误。

服务器将转储表的`CREATE`定义发送给**mysqldump**，后者将它们写入`.sql`文件。因此，这些文件的所有者是执行**mysqldump**的用户。

最好只在本地服务器上使用`--tab`进行转储。如果在远程服务器上使用它，那么`--tab`目录必须同时存在于本地和远程主机上，并且`.txt`文件由服务器在远程目录（在服务器主机上）中写入，而`.sql`文件由**mysqldump**在本地目录（在客户端主机上）中写入。

对于**mysqldump --tab**，服务器默认将表数据写入`.txt`文件，每行一个行，列值之间用制表符分隔，列值周围没有引号，换行符作为行终止符。（这些是与`SELECT ... INTO OUTFILE`相同的默认值。）

为了使数据文件以不同格式编写，**mysqldump**支持以下选项：

+   `--fields-terminated-by=*`str`*`

    用于分隔列值的字符串（默认：制表符）。

+   `--fields-enclosed-by=*`char`*`

    用于封装列值的字符（默认：无字符）。

+   `--fields-optionally-enclosed-by=*`char`*`

    用于封装非数字列值的字符（默认：无字符）。

+   `--fields-escaped-by=*`char`*`

    用于转义特殊字符的字符（默认：不转义）。

+   `--lines-terminated-by=*`str`*`

    行终止字符串（默认：换行符）。

根据您为这些选项中的任何一个指定的值，可能需要在命令行上适当地引用或转义该值。或者，使用十六进制表示值。假设您希望**mysqldump**在双引号内引用列值。为此，请将双引号指定为`--fields-enclosed-by`选项的值。但是，此字符通常对命令解释器特殊，必须特殊处理。例如，在 Unix 上，您可以这样引用双引号：

```sql
--fields-enclosed-by='"'
```

在任何平台上，您都可以以十六进制指定值：

```sql
--fields-enclosed-by=0x22
```

通常会一起使用几个数据格式选项。例如，要以逗号分隔值格式转储表，并以回车/换行对（`\r\n`）终止行，请使用以下命令（在一行上输入）：

```sql
$> mysqldump --tab=/tmp --fields-terminated-by=,
         --fields-enclosed-by='"' --lines-terminated-by=0x0d0a db1
```

如果您使用任何数据格式选项来转储表数据，则在以后重新加载数据文件时，需要指定相同的格式，以确保正确解释文件内容。
