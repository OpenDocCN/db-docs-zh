# 9.4.2 重新加载 SQL 格式备份

> 原文：[`dev.mysql.com/doc/refman/8.0/en/reloading-sql-format-dumps.html`](https://dev.mysql.com/doc/refman/8.0/en/reloading-sql-format-dumps.html)

要重新加载由**mysqldump**编写的包含 SQL 语句的转储文件，请将其用作**mysql**客户端的输入。如果转储文件是由**mysqldump**使用`--all-databases`或`--databases`选项创建的，则其中包含`CREATE DATABASE`和`USE`语句，因此无需指定默认数据库来加载数据：

```sql
$> mysql < dump.sql
```

或者，从**mysql**内部使用`source`命令：

```sql
mysql> source dump.sql
```

如果文件是一个不包含`CREATE DATABASE`和`USE`语句的单个数据库转储文件，请首先创建数据库（如果需要）：

```sql
$> mysqladmin create db1
```

然后在加载转储文件时指定数据库名称：

```sql
$> mysql db1 < dump.sql
```

或者，从**mysql**内部创建数据库，将其选择为默认数据库，并加载转储文件：

```sql
mysql> CREATE DATABASE IF NOT EXISTS db1;
mysql> USE db1;
mysql> source dump.sql
```

注意

对于 Windows PowerShell 用户：由于"<"字符在 PowerShell 中保留供将来使用，因此需要另一种方法，例如使用引号`cmd.exe /c "mysql < dump.sql"`。
