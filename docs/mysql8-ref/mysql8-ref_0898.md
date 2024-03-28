# 15.1.12 CREATE DATABASE Statement

> 原文：[`dev.mysql.com/doc/refman/8.0/en/create-database.html`](https://dev.mysql.com/doc/refman/8.0/en/create-database.html)

```sql
CREATE {DATABASE | SCHEMA} [IF NOT EXISTS] *db_name*
    [*create_option*] ...

*create_option*: [DEFAULT] {
    CHARACTER SET [=] *charset_name*
  | COLLATE [=] *collation_name*
  | ENCRYPTION [=] {'Y' | 'N'}
}
```

`CREATE DATABASE`使用给定的名称创建数据库。要使用此语句，您需要数据库的`CREATE`权限。`CREATE SCHEMA`是`CREATE DATABASE`的同义词。

如果数据库存在且您未指定`IF NOT EXISTS`，则会发生错误。

在具有活动`LOCK TABLES`语句的会话中，不允许使用`CREATE DATABASE`。

每个*`create_option`*指定一个数据库特性。数据库特性存储在数据字典中。

+   `CHARACTER SET`选项指定默认的数据库字符集。`COLLATE`选项指定默认的数据库校对规则。有关字符集和校对规则名称的信息，请参见 Chapter 12, *Character Sets, Collations, Unicode*。

    要查看可用的字符集和校对规则，请分别使用`SHOW CHARACTER SET`和`SHOW COLLATION`语句。参见 Section 15.7.7.3, “SHOW CHARACTER SET Statement”和 Section 15.7.7.4, “SHOW COLLATION Statement”。

+   `ENCRYPTION`选项是在 MySQL 8.0.16 中引入的，定义了默认的数据库加密，该加密会被创建在数据库中的表继承。允许的值为`'Y'`（启用加密）和`'N'`（禁用加密）。如果未指定`ENCRYPTION`选项，则`default_table_encryption`系统变量的值定义了默认的数据库加密。如果启用了`table_encryption_privilege_check`系统变量，则需要`TABLE_ENCRYPTION_ADMIN`权限来指定与`default_table_encryption`设置不同的默认加密设置。有关更多信息，请参见为模式和通用表空间定义加密默认值。

在 MySQL 中，数据库被实现为一个包含与数据库中表对应的文件的目录。因为在初始创建数据库时数据库中没有表，所以`CREATE DATABASE`语句只会在 MySQL 数据目录下创建一个目录。有关可接受数据库名称的规则，请参阅第 11.2 节，“模式对象名称”。如果数据库名称包含特殊字符，则数据库目录的名称将包含这些字符的编码版本，如第 11.2.4 节，“标识符映射到文件名”中所述。

在 MySQL 8.0 中，通过手动在数据目录下创建目录（例如，使用**mkdir**）来创建数据库目录是不受支持的。

当你创建一个数据库时，让服务器管理目录和其中的文件。直接操作数据库目录和文件可能会导致不一致和意外结果。

MySQL 对数据库数量没有限制。底层文件系统可能对目录数量有限制。

您也可以使用**mysqladmin**程序来创建数据库。请参阅第 6.5.2 节，“mysqladmin — MySQL 服务器管理程序”。
