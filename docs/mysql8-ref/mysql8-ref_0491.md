# 9.4.1 使用 mysqldump 以 SQL 格式转储数据

> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysqldump-sql-format.html`](https://dev.mysql.com/doc/refman/8.0/en/mysqldump-sql-format.html)

本节描述如何使用**mysqldump**创建 SQL 格式的转储文件。有关重新加载此类转储文件的信息，请参见 Section 9.4.2, “Reloading SQL-Format Backups”。

默认情况下，**mysqldump**将信息写入标准输出作为 SQL 语句。您可以将输出保存在文件中：

```sql
$> mysqldump [*arguments*] > *file_name*
```

要转储所有数据库，请使用`--all-databases`选项调用**mysqldump**：

```sql
$> mysqldump --all-databases > dump.sql
```

要仅转储特定数据库，请在命令行上命名它们并使用`--databases`选项：

```sql
$> mysqldump --databases db1 db2 db3 > dump.sql
```

`--databases`选项导致命令行上的所有名称被视为数据库名称。没有此选项，**mysqldump**将第一个名称视为数据库名称，后续名称视为表名称。

使用`--all-databases`或`--databases`，**mysqldump**在转储输出之前为每个数据库写入`CREATE DATABASE`和`USE`语句。这确保在重新加载转储文件时，如果数据库不存在，则创建每个数据库并将其设置为默认数据库，因此数据库内容将加载到与其来源相同的数据库中。如果要导致转储文件在重新创建数据库之前强制删除每个数据库，请同时使用`--add-drop-database`选项。在这种情况下，**mysqldump**在每个`CREATE DATABASE`语句之前写入一个`DROP DATABASE`语句。

要转储单个数据库，请在命令行上命名它：

```sql
$> mysqldump --databases test > dump.sql
```

在单个数据库的情况下，可以省略`--databases`选项：

```sql
$> mysqldump test > dump.sql
```

两个前述命令之间的区别在于，没有`--databases`，转储输出不包含`CREATE DATABASE`或`USE`语句。这有几个影响：

+   当重新加载转储文件时，必须指定一个默认数据库名称，以便服务器知道要重新加载哪个数据库。

+   对于重新加载，可以指定与原始名称不同的数据库名称，这使您可以将数据重新加载到不同的数据库中。

+   如果要重新加载的数据库不存在，必须首先创建它。

+   因为输出不包含`CREATE DATABASE`语句，`--add-drop-database`选项没有效果。如果使用它，将不会生成`DROP DATABASE`语句。

为了仅从数据库中导出特定表，需要在命令行中跟随数据库名称命名这些表：

```sql
$> mysqldump test t1 t3 t7 > dump.sql
```

默认情况下，如果在创建转储文件的服务器上使用了 GTIDs（`gtid_mode=ON`），**mysqldump**在输出中包含一个`SET @@GLOBAL.gtid_purged`语句，将源服务器上的`gtid_executed`集合中的 GTIDs 添加到目标服务器上的`gtid_purged`集合中。如果仅转储特定数据库或表，重要的是要注意由**mysqldump**包含的值包括源服务器上`gtid_executed`集合中的所有事务的 GTIDs，即使这些事务更改了数据库的被抑制部分，或者服务器上未包含在部分转储中的其他数据库。如果您只在目标服务器上重放一个部分转储文件，额外的 GTIDs 不会对该服务器的未来操作造成任何问题。但是，如果在目标服务器上重放包含相同 GTIDs 的第二个转储文件（例如，来自同一源服务器的另一个部分转储），第二个转储文件中的任何`SET @@GLOBAL.gtid_purged`语句将失败。为避免此问题，要么将**mysqldump**选项`--set-gtid-purged`设置为`OFF`或`COMMENTED`，以在输出第二个转储文件时不包含活动的`SET @@GLOBAL.gtid_purged`语句，要么在重放转储文件之前手动删除该语句。
