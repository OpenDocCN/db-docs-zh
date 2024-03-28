# 12.3.4 表字符集和排序规则

> 原文：[`dev.mysql.com/doc/refman/8.0/en/charset-table.html`](https://dev.mysql.com/doc/refman/8.0/en/charset-table.html)

每个表都有一个表字符集和一个表排序规则。`CREATE TABLE`和`ALTER TABLE`语句有可选子句，用于指定表字符集和排序规则：

```sql
CREATE TABLE *tbl_name* (*column_list*)
    [[DEFAULT] CHARACTER SET *charset_name*]
    [COLLATE *collation_name*]]

ALTER TABLE *tbl_name*
    [[DEFAULT] CHARACTER SET *charset_name*]
    [COLLATE *collation_name*]
```

示例：

```sql
CREATE TABLE t1 ( ... )
CHARACTER SET latin1 COLLATE latin1_danish_ci;
```

MySQL 选择表字符集和排序规则的方式如下：

+   如果同时指定了`CHARACTER SET *`charset_name`*`和`COLLATE *`collation_name`*`，则使用字符集*`charset_name`*和排序规则*`collation_name`*。

+   如果指定了`CHARACTER SET *`charset_name`*`，但没有指定`COLLATE`，则使用字符集*`charset_name`*及其默认排序规则。要查看每个字符集的默认排序规则，请使用`SHOW CHARACTER SET`语句或查询`INFORMATION_SCHEMA` `CHARACTER_SETS`表。

+   如果指定了`COLLATE *`collation_name`*`，但没有指定`CHARACTER SET`，则使用与*`collation_name`*相关联的字符集和排序规则*`collation_name`*。

+   否则（未指定`CHARACTER SET`或`COLLATE`），则使用数据库字符集和排序规则。

如果在单独的列定义中未指定列字符集和排序规则，则表字符集和排序规则将用作列定义的默认值。表字符集和排序规则是 MySQL 的扩展；标准 SQL 中没有这样的东西。
