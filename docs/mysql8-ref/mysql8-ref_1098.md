> 原文：[`dev.mysql.com/doc/refman/8.0/en/show-index.html`](https://dev.mysql.com/doc/refman/8.0/en/show-index.html)

#### 15.7.7.22 显示索引语句

```sql
SHOW [EXTENDED] {INDEX | INDEXES | KEYS}
    {FROM | IN} *tbl_name*
    [{FROM | IN} *db_name*]
    [WHERE *expr*]
```

`显示索引`返回表索引信息。格式类似于 ODBC 中的`SQLStatistics`调用。此语句对表中的任何列都需要一些特权。

```sql
mysql> SHOW INDEX FROM City\G
*************************** 1\. row ***************************
        Table: city
   Non_unique: 0
     Key_name: PRIMARY
 Seq_in_index: 1
  Column_name: ID
    Collation: A
  Cardinality: 4188
     Sub_part: NULL
       Packed: NULL
         Null:
   Index_type: BTREE
      Comment:
Index_comment:
      Visible: YES
   Expression: NULL
*************************** 2\. row ***************************
        Table: city
   Non_unique: 1
     Key_name: CountryCode
 Seq_in_index: 1
  Column_name: CountryCode
    Collation: A
  Cardinality: 232
     Sub_part: NULL
       Packed: NULL
         Null:
   Index_type: BTREE
      Comment:
Index_comment:
      Visible: YES
   Expression: NULL
```

作为`*`tbl_name`* FROM *`db_name`*`语法的替代方案是*`db_name`*.*`tbl_name`*。这两个语句是等效的：

```sql
SHOW INDEX FROM mytable FROM mydb;
SHOW INDEX FROM mydb.mytable;
```

可选的`EXTENDED`关键字导致输出包括 MySQL 内部使用但用户无法访问的隐藏索引的信息。

可以使用`WHERE`子句选择使用更一般条件的行，如第 28.8 节“SHOW 语句的扩展”中所讨论的。

`显示索引`返回以下字段：

+   `表`

    表的名称。

+   `非唯一`

    如果索引不能包含重复项，则为 0，如果可以，则为 1。

+   `键名`

    索引的名称。如果索引是主键，则名称始终为`PRIMARY`。

+   `Seq_in_index`

    索引中的列序号，从 1 开始。

+   `列名`

    列名。另请参阅`Expression`列的描述。

+   `排序规则`

    列在索引中的排序方式。这可以是`A`（升序）、`D`（降序）或`NULL`（未排序）的值。

+   `基数`

    索引中唯一值的估计数量。要更新此数字，请运行`ANALYZE TABLE`或（对于`MyISAM`表）**myisamchk -a**。

    `基数`是根据存储为整数的统计数据计算的，因此即使对于小表，该值也不一定是精确的。基数越高，MySQL 在执行连接时使用索引的可能性就越大。

+   `子部分`

    索引前缀。也就是，如果列仅部分索引，则索引字符数，如果整个列被索引，则为`NULL`。

    注意

    前缀*限制*以字节为单位。但是，在`CREATE TABLE`、`ALTER TABLE`和`CREATE INDEX`语句中的索引规范中，对于非二进制字符串类型（`CHAR`、`VARCHAR`、`TEXT`），解释为字符数，对于二进制字符串类型（`BINARY`、`VARBINARY`、`BLOB`），解释为字节数。在为使用多字节字符集的非二进制字符串列指定前缀长度时，请考虑这一点。

    有关索引前缀的其他信息，请参见第 10.3.5 节，“列索引”和第 15.1.15 节，“CREATE INDEX Statement”。

+   `Packed`

    指示键是如何打包的。如果不是，则为`NULL`。

+   `Null`

    包含`YES`表示列可能包含`NULL`值，`''`表示不包含。

+   `Index_type`

    使用的索引方法（`BTREE`、`FULLTEXT`、`HASH`、`RTREE`）。

+   `Comment`

    有关索引未在其自己的列中描述的信息，例如如果索引已禁用，则为`disabled`。

+   `Index_comment`

    在创建索引时使用`COMMENT`属性提供的任何注释。

+   `Visible`

    索引是否对优化器可见。请参见第 10.3.12 节，“不可见索引”。

+   `Expression`

    MySQL 8.0.13 及更高版本支持功能键部分（参见功能键部分表中获取。请参见第 28.3.34 节，“INFORMATION_SCHEMA STATISTICS Table”。有关隐藏索引的扩展信息仅可使用`SHOW EXTENDED INDEX`获得；无法从`STATISTICS`表中获取。

您可以使用**mysqlshow -k *`db_name`* *`tbl_name`***命令列出表的索引。

在 MySQL 8.0.30 及更高版本中，默认情况下，`SHOW INDEX` 包括表的生成的不可见键。您可以通过设置`show_gipk_in_create_table_and_information_schema = OFF`来抑制该信息在语句输出中的显示。更多信息，请参见 Section 15.1.20.11, “Generated Invisible Primary Keys”。
