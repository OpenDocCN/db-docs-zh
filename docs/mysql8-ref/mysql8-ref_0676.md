# 12.3.3 数据库字符集和排序规则

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-database.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-database.html)

每个数据库都有一个数据库字符集和一个数据库排序规则。`CREATE DATABASE`和`ALTER DATABASE`语句具有用于指定数据库字符集和排序规则的可选子句：

```sql
CREATE DATABASE *db_name*
    [[DEFAULT] CHARACTER SET *charset_name*]
    [[DEFAULT] COLLATE *collation_name*]

ALTER DATABASE *db_name*
    [[DEFAULT] CHARACTER SET *charset_name*]
    [[DEFAULT] COLLATE *collation_name*]
```

关键字`SCHEMA`可以代替`DATABASE`。

`CHARACTER SET`和`COLLATE`子句使得在同一 MySQL 服务器上创建具有不同字符集和排序规则的数据库成为可能。

数据库选项存储在数据字典中，可以通过检查信息模式`SCHEMATA`表来查看。

示例：

```sql
CREATE DATABASE *db_name* CHARACTER SET latin1 COLLATE latin1_swedish_ci;
```

MySQL 按以下方式选择数据库字符集和数据库排序规则：

+   如果同时指定了`CHARACTER SET *`charset_name`*`和`COLLATE *`collation_name`*`，则使用字符集*`charset_name`*和排序规则*`collation_name`*。

+   如果指定了`CHARACTER SET *`charset_name`*`而没有指定`COLLATE`，则使用字符集*`charset_name`*及其默认排序规则。要查看每个字符集的默认排序规则，请使用`SHOW CHARACTER SET`语句或查询`INFORMATION_SCHEMA` `CHARACTER_SETS`表。

+   如果指定了`COLLATE *`collation_name`*`而没有指定`CHARACTER SET`，则使用与*`collation_name`*关联的字符集和排序规则*`collation_name`*。

+   否则（既未指定`CHARACTER SET`也未指定`COLLATE`），则使用服务器字符集和服务器排序规则。

可以从`character_set_database`和`collation_database`系统变量的值确定默认数据库的字符集和排序规则。每当默认数据库更改时，服务器会设置这些变量的值。如果没有默认数据库，则这些变量的值与相应的服务器级系统变量`character_set_server`和`collation_server`相同。

要查看给定数据库的默认字符集和排序规则，请使用以下语句：

```sql
USE *db_name*;
SELECT @@character_set_database, @@collation_database;
```

或者，要显示值而不更改默认数据库：

```sql
SELECT DEFAULT_CHARACTER_SET_NAME, DEFAULT_COLLATION_NAME
FROM INFORMATION_SCHEMA.SCHEMATA WHERE SCHEMA_NAME = '*db_name*';
```

数据库字符集和排序规则影响服务器操作的这些方面：

+   对于`CREATE TABLE`语句，如果未指定表的字符集和校对规则，则数据库字符集和校对规则将作为表定义的默认值。要覆盖此设置，请提供明确的`CHARACTER SET`和`COLLATE`表选项。

+   对于包含没有`CHARACTER SET`子句的`LOAD DATA`语句，服务器将使用`character_set_database`系统变量指示的字符集来解释文件中的信息。要覆盖此设置，请提供明确的`CHARACTER SET`子句。

+   对于存储过程（procedures）和函数（functions），在创建过程中使用的数据库字符集和校对规则将作为字符数据参数的字符集和校对规则，如果声明中未包含`CHARACTER SET`或`COLLATE`属性。要覆盖此设置，请明确提供`CHARACTER SET`和`COLLATE`。
