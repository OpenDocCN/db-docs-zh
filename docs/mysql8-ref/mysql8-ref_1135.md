# 16.1 数据字典模式

> 原文：[`dev.mysql.com/doc/refman/8.0/en/data-dictionary-schema.html`](https://dev.mysql.com/doc/refman/8.0/en/data-dictionary-schema.html)

数据字典表受到保护，只能在 MySQL 的调试版本中访问。但是，MySQL 支持通过`INFORMATION_SCHEMA`表和`SHOW`语句访问存储在数据字典表中的数据。有关组成数据字典的表的概述，请参阅数据字典表。

MySQL 系统表仍然存在于 MySQL 8.0 中，并且可以通过在`mysql`系统数据库上发出`SHOW TABLES`语句来查看。通常，MySQL 数据字典表和系统表之间的区别在于数据字典表包含执行 SQL 查询所需的元数据，而系统表包含辅助数据，如时区和帮助信息。 MySQL 系统表和数据字典表在升级方式上也有所不同。MySQL 服务器管理数据字典升级。请参阅数据字典如何升级。升级 MySQL 系统表需要运行完整的 MySQL 升级过程。请参阅第 3.4 节，“MySQL 升级过程升级了什么”。

### 数据字典如何升级

MySQL 的新版本可能包含对数据字典表定义的更改。这些更改存在于新安装的 MySQL 版本中，但在执行 MySQL 二进制文件的就地升级时，更改将在使用新二进制文件重新启动 MySQL 服务器时应用。在启动时，服务器的数据字典版本将与数据字典中存储的版本信息进行比较，以确定是否应该升级数据字典表。如果需要升级并且受支持，服务器将使用更新的定义创建数据字典表，将持久化的元数据复制到新表中，以原子方式用新表替换旧表，并重新初始化数据字典。如果不需要升级，则启动将继续而不更新数据字典表。

数据字典表的升级是一个原子操作，这意味着所有必要的数据字典表都会被升级，否则操作将失败。如果升级操作失败，服务器启动将以错误结束。在这种情况下，可以使用旧的服务器二进制文件和旧的数据目录来启动服务器。当再次使用新的服务器二进制文件启动服务器时，数据字典升级将重新尝试。

通常，在成功升级数据字典表后，不可能使用旧的服务器二进制文件重新启动服务器。因此，在升级数据字典表后，不支持将 MySQL 服务器二进制文件降级到先前的 MySQL 版本。

**mysqld** `--no-dd-upgrade` 选项可用于防止在启动时自动升级数据字典表。当指定 `--no-dd-upgrade` 时，如果服务器发现服务器的数据字典版本与数据字典中存储的版本不同，则启动将失败，并显示禁止数据字典升级的错误。

### 使用 MySQL 的调试版本查看数据字典表

数据字典表默认受保护，但可以通过使用带有调试支持的 MySQL 编译（使用`-DWITH_DEBUG=1` **CMake** 选项）并指定`+d,skip_dd_table_access_check` `debug` 选项和修饰符来访问。有关编译调试版本的信息，请参见 Section 7.9.1.1, “Compiling MySQL for Debugging”。

警告

不建议直接修改或写入数据字典表，这可能导致您的 MySQL 实例无法运行。

在使用带有调试支持的 MySQL 编译后，使用此 `SET` 语句使数据字典表对**mysql** 客户端会话可见：

```sql
mysql> SET SESSION debug='+d,skip_dd_table_access_check';
```

使用此查询检索数据字典表的列表：

```sql
mysql> SELECT name, schema_id, hidden, type FROM mysql.tables where schema_id=1 AND hidden='System';
```

使用 `SHOW CREATE TABLE` 查看数据字典表定义。例如：

```sql
mysql> SHOW CREATE TABLE mysql.catalogs\G
```
