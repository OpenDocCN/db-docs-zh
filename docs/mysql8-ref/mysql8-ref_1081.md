> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-columns.html`](https://dev.mysql.com/doc/refman/8.0/en/show-columns.html)

#### 15.7.7.5 显示列语句

```sql
SHOW [EXTENDED] [FULL] {COLUMNS | FIELDS}
    {FROM | IN} *tbl_name*
    [{FROM | IN} *db_name*]
    [LIKE '*pattern*' | WHERE *expr*]
```

`显示列`显示给定表中列的信息。它也适用于视图。`显示列`仅显示您具有某些权限的列的信息。

```sql
mysql> SHOW COLUMNS FROM City;
+-------------+----------+------+-----+---------+----------------+
| Field       | Type     | Null | Key | Default | Extra          |
+-------------+----------+------+-----+---------+----------------+
| ID          | int(11)  | NO   | PRI | NULL    | auto_increment |
| Name        | char(35) | NO   |     |         |                |
| CountryCode | char(3)  | NO   | MUL |         |                |
| District    | char(20) | NO   |     |         |                |
| Population  | int(11)  | NO   |     | 0       |                |
+-------------+----------+------+-----+---------+----------------+
```

`*tbl_name* FROM *db_name*`语法的替代方案是*`db_name.tbl_name`*。这两个语句是等效的：

```sql
SHOW COLUMNS FROM mytable FROM mydb;
SHOW COLUMNS FROM mydb.mytable;
```

可选的`EXTENDED`关键字导致输出包括关于 MySQL 内部使用但用户无法访问的隐藏列的信息。

可选的`FULL`关键字导致输出包括列排序规则和注释，以及您对每列的权限。

如果存在`LIKE`子句，则指示要匹配的列名。可以使用`WHERE`子句以更一般的条件选择行，如第 28.8 节“SHOW 语句的扩展”中所讨论的。

数据类型可能与您根据`CREATE TABLE`语句期望的不同，因为 MySQL 有时在创建或更改表时会更改数据类型。发生这种情况的条件在第 15.1.20.7 节“静默列规范更改”中有描述。

`显示列`为每个表列显示以下数值：

+   `Field`

    列的名称。

+   `Type`

    列数据类型。

+   `Collation`

    非二进制字符串列的排序规则，或其他列的`NULL`值。仅当使用`FULL`关键字时才显示此值。

+   `Null`

    列的可空性。如果列中可以存储`NULL`值，则该值为`YES`，否则为`NO`。

+   `Key`

    列是否被索引：

    +   如果`Key`为空，则该列要么未被索引，要么仅作为多列非唯一索引中的次要列被索引。

    +   如果`Key`为`PRI`，则该列是`PRIMARY KEY`或是多列`PRIMARY KEY`中的一列。

    +   如果`Key`为`UNI`，则该列是`UNIQUE`索引的第一列。（`UNIQUE`索引允许多个`NULL`值，但您可以通过检查`Null`字段来确定该列是否允许`NULL`。）

    +   如果`Key`为`MUL`，则该列是非唯一索引的第一列，在该索引中允许列中出现给定值的多个实例。

    如果多个`Key`值适用于表的某一列，则`Key`按照`PRI`，`UNI`，`MUL`的顺序显示具有最高优先级的值。

    如果`UNIQUE`索引不能包含`NULL`值且表中没有`PRIMARY KEY`，则`UNIQUE`索引可能显示为`PRI`。如果几列形成复合`UNIQUE`索引，则`UNIQUE`索引可能显示为`MUL`；尽管列的组合是唯一的，但每列仍然可以包含给定值的多个出现。

+   `默认`

    列的默认值。如果列具有显式默认值为`NULL`，或者列定义中不包含`DEFAULT`子句，则为`NULL`。

+   `额外`

    有关给定列的任何其他可用信息。在以下情况下，该值不为空：

    +   对于具有`AUTO_INCREMENT`属性的列，显示`auto_increment`。

    +   对于具有`ON UPDATE CURRENT_TIMESTAMP`属性的`TIMESTAMP`或`DATETIME`列，显示`on update CURRENT_TIMESTAMP`。

    +   用于生成列的`VIRTUAL GENERATED`或`STORED GENERATED`。

    +   对于具有表达式默认值的列，使用`DEFAULT_GENERATED`。

+   `权限`

    您对该列的权限。仅当使用`FULL`关键字时才显示此值。

+   `注释`

    列定义中包含的任何注释。仅当使用`FULL`关键字时才显示此值。

表列信息也可以从`INFORMATION_SCHEMA`的`COLUMNS`表中获取。请参阅第 28.3.8 节，“The INFORMATION_SCHEMA COLUMNS Table”。有关隐藏列的扩展信息仅可使用`SHOW EXTENDED COLUMNS`获得；无法从`COLUMNS`表中获取。

您可以使用**mysqlshow *`db_name`* *`tbl_name`***命令列出表的列。

`DESCRIBE`语句提供类似于`SHOW COLUMNS`的信息。请参阅第 15.8.1 节，“DESCRIBE Statement”。

`SHOW CREATE TABLE`，`SHOW TABLE STATUS`和`SHOW INDEX`语句还提供有关表的信息。请参阅第 15.7.7 节，“SHOW Statements”。

在 MySQL 8.0.30 及更高版本中，默认情况下，`SHOW COLUMNS` 包括表的生成的不可见主键。您可以通过设置 `show_gipk_in_create_table_and_information_schema = OFF` 来使此信息在语句输出中被抑制。更多信息，请参见 Section 15.1.20.11, “Generated Invisible Primary Keys”。
