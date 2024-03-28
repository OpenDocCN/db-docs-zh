# 5.4 获取有关数据库和表的信息

> 原文：[`dev.mysql.com/doc/refman/8.0/en/getting-information.html`](https://dev.mysql.com/doc/refman/8.0/en/getting-information.html)

如果您忘记了数据库或表的名称，或者给定表的结构是什么（例如，其列叫什么）？MySQL 通过几个语句解决了这个问题，这些语句提供有关其支持的数据库和表的信息。

您之前已经看到`SHOW DATABASES`，它列出了服务器管理的数据库。要找出当前选择的数据库是哪个，请使用`DATABASE()`函数：

```sql
mysql> SELECT DATABASE();
+------------+
| DATABASE() |
+------------+
| menagerie  |
+------------+
```

如果您尚未选择任何数据库，则结果为`NULL`。

要查找默认数据库包含哪些表（例如，当您不确定表名时），请使用此语句：

```sql
mysql> SHOW TABLES;
+---------------------+
| Tables_in_menagerie |
+---------------------+
| event               |
| pet                 |
+---------------------+
```

该语句生成的输出中列的名称始终为`Tables_in_*`db_name`*`，其中*`db_name`*是数据库的名称。有关更多信息，请参阅 Section 15.7.7.39, “SHOW TABLES Statement”。

如果您想了解表的结构，`DESCRIBE`语句很有用；它显示表的每个列的信息：

```sql
mysql> DESCRIBE pet;
+---------+-------------+------+-----+---------+-------+
| Field   | Type        | Null | Key | Default | Extra |
+---------+-------------+------+-----+---------+-------+
| name    | varchar(20) | YES  |     | NULL    |       |
| owner   | varchar(20) | YES  |     | NULL    |       |
| species | varchar(20) | YES  |     | NULL    |       |
| sex     | char(1)     | YES  |     | NULL    |       |
| birth   | date        | YES  |     | NULL    |       |
| death   | date        | YES  |     | NULL    |       |
+---------+-------------+------+-----+---------+-------+
```

`Field`表示列名，`Type`是列的数据类型，`NULL`指示列是否可以包含`NULL`值，`Key`指示列是否已索引，`Default`指定列的默认值。`Extra`显示有关列的特殊信息：如果使用`AUTO_INCREMENT`选项创建列，则值为`auto_increment`而不是空。

`DESC`是`DESCRIBE`语句的简写形式。有关更多信息，请参阅 Section 15.8.1, “DESCRIBE Statement”。

您可以使用`SHOW CREATE TABLE`语句获取创建现有表所需的`CREATE TABLE`语句。请参阅 Section 15.7.7.10, “SHOW CREATE TABLE Statement”。

如果表上有索引，则`SHOW INDEX FROM *`tbl_name`*`会提供有关它们的信息。有关此语句的更多信息，请参阅 Section 15.7.7.22, “SHOW INDEX Statement”。
