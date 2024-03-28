# 3.14 重建或修复表或索引

> 原文：[`dev.mysql.com/doc/refman/8.0/en/rebuilding-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/rebuilding-tables.html)

本节描述了如何重建或修复表或索引，可能是由于：

+   MySQL 处理数据类型或字符集的更改。例如，校对错误可能已经被纠正，需要重建表以更新使用该校对的字符列的索引。

+   `CHECK TABLE` 报告的需要表修复或升级，**mysqlcheck** 或 **mysql_upgrade**。

重建表的方法包括：

+   转储和重新加载方法

+   ALTER TABLE 方法

+   REPAIR TABLE 方法

### 转储和重新加载方法

如果您因为不同版本的 MySQL 在二进制（原地）升级或降级后无法处理它们而需要重建表，您必须使用转储和重新加载的方法。在使用原始版本的 MySQL 升级或降级*之前*转储表。然后在升级或降级*之后*重新加载表。

如果您仅使用转储和重新加载方法来重建表以重建索引，您可以在升级或降级之前或之后执行转储。重新加载仍然必须在之后进行。

如果您需要重建一个 `InnoDB` 表，因为 `CHECK TABLE` 操作指示需要进行表升级，请使用 **mysqldump** 创建一个转储文件，然后使用 **mysql** 重新加载文件。如果 `CHECK TABLE` 操作指示存在损坏或导致 `InnoDB` 失败，请参考 第 17.21.3 节，“强制 InnoDB 恢复” 了解如何使用 `innodb_force_recovery` 选项重新启动 `InnoDB`。要了解 `CHECK TABLE` 可能遇到的问题类型，请参考 第 15.7.3.2 节，“CHECK TABLE 语句” 中的 `InnoDB` 注释。

要通过转储和重新加载来重建表，使用 **mysqldump** 创建一个转储文件，然后使用 **mysql** 重新加载文件：

```sql
mysqldump *db_name* t1 > dump.sql
mysql *db_name* < dump.sql
```

要重建单个数据库中的所有表格，请指定数据库名称，不需要跟随任何表格名称：

```sql
mysqldump *db_name* > dump.sql
mysql *db_name* < dump.sql
```

要重建所有数据库中的所有表格，请使用`--all-databases`选项：

```sql
mysqldump --all-databases > dump.sql
mysql < dump.sql
```

### ALTER TABLE 方法

要使用`ALTER TABLE`重新构建表格，请使用“null”修改；也就是说，使用一个`ALTER TABLE`语句“更改”表格以使用它已经具有的存储引擎。例如，如果`t1`是一个`InnoDB`表格，使用以下语句：

```sql
ALTER TABLE t1 ENGINE = InnoDB;
```

如果不确定在`ALTER TABLE`语句中指定哪个存储引擎，请使用`SHOW CREATE TABLE`来显示表格定义。

### REPAIR TABLE 方法

`REPAIR TABLE`方法仅适用于`MyISAM`、`ARCHIVE`和`CSV`表格。

如果表格检查操作指示存在损坏或需要升级，则可以使用`REPAIR TABLE`。例如，要修复一个`MyISAM`表格，请使用以下语句：

```sql
REPAIR TABLE t1;
```

**mysqlcheck --repair**提供了对`REPAIR TABLE`语句的命令行访问。这可以是一个更方便的修复表格的方法，因为您可以使用`--databases`或`--all-databases`选项来修复特定数据库中的所有表格或所有数据库中的所有表格：

```sql
mysqlcheck --repair --databases *db_name* ...
mysqlcheck --repair --all-databases
```
