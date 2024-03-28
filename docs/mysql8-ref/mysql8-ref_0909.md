> 原文：[`dev.mysql.com/doc/refman/8.0/en/create-table-like.html`](https://dev.mysql.com/doc/refman/8.0/en/create-table-like.html)

#### 15.1.20.3 CREATE TABLE ... LIKE 语句

使用`CREATE TABLE ... LIKE`根据另一个表的定义创建一个空表，包括原始表中定义的任何列属性和索引：

```sql
CREATE TABLE *new_tbl* LIKE *orig_tbl*;
```

复制是使用与原始表相同版本的表存储格式创建的。需要在原始表上具有`SELECT`权限。

`LIKE`仅适用于基本表，不适用于视图。

重要提示

在执行`LOCK TABLES`语句时，不能执行`CREATE TABLE`或`CREATE TABLE ... LIKE`。

`CREATE TABLE ... LIKE`执行与`CREATE TABLE`相同的检查。这意味着如果当前的 SQL 模式与创建原始表时生效的模式不同，表定义可能被认为对新模式无效并导致语句失败。

对于`CREATE TABLE ... LIKE`，目标表保留原始表中的生成列信息。

对于`CREATE TABLE ... LIKE`，目标表保留原始表中的表达式默认值。

对于`CREATE TABLE ... LIKE`，目标表保留原始表中的`CHECK`约束，只是所有约束名称都是自动生成的。

`CREATE TABLE ... LIKE`不保留为原始表指定的任何`DATA DIRECTORY`或`INDEX DIRECTORY`表选项，也不保留任何外键定义。

如果原始表是临时表，则`CREATE TABLE ... LIKE`不保留`TEMPORARY`。要创建一个`TEMPORARY`目标表，请使用`CREATE TEMPORARY TABLE ... LIKE`。

在`mysql`表空间、`InnoDB`系统表空间（`innodb_system`）或通用表空间中创建的表包括表定义中的`TABLESPACE`属性，该属性定义了表所在的表空间。由于临时回归，`CREATE TABLE ... LIKE`保留了`TABLESPACE`属性，并在定义的表空间中创建表，而不管`innodb_file_per_table`设置如何。为了在基于此类表的定义创建空表时避免`TABLESPACE`属性，请改用以下语法：

```sql
CREATE TABLE *new_tbl* SELECT * FROM *orig_tbl* LIMIT 0;
```

`CREATE TABLE ... LIKE`操作将所有`ENGINE_ATTRIBUTE`和`SECONDARY_ENGINE_ATTRIBUTE`值应用于新表。
