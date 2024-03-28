# 18.7 MERGE 存储引擎

> 原文：[`dev.mysql.com/doc/refman/8.0/en/merge-storage-engine.html`](https://dev.mysql.com/doc/refman/8.0/en/merge-storage-engine.html)

18.7.1 MERGE 表优缺点

18.7.2 MERGE 表问题

`MERGE`存储引擎，也称为`MRG_MyISAM`引擎，是一组相同的`MyISAM`表，可以作为一个使用。“相同”意味着所有表具有相同的列数据类型和索引信息。您不能合并列在不同顺序中列出、相应列中的数据类型不完全相同或索引顺序不同的`MyISAM`表。但是，任何或所有`MyISAM`表都可以使用**myisampack**进行压缩。请参见第 6.6.6 节，“myisampack — 生成压缩的只读 MyISAM 表”。这些表之间的差异并不重要：

+   相应列和索引的名称可能不同。

+   表、列和索引的注释可能不同。

+   表选项如`AVG_ROW_LENGTH`、`MAX_ROWS`或`PACK_KEYS`可能不同。

`MERGE`表的替代方案是分区表，它将单个表的分区存储在单独的文件中，并使某些操作更有效。有关更多信息，请参见第二十六章，*分区*。

创建`MERGE`表时，MySQL 在磁盘上创建一个包含应作为一个的底层`MyISAM`表名称的`.MRG`文件。`MERGE`表的表格式存储在 MySQL 数据字典中。底层表不必与`MERGE`表在同一个数据库中。

您可以在`MERGE`表上使用`SELECT`、`DELETE`、`UPDATE`和`INSERT`。您必须对映射到`MERGE`表的`MyISAM`表具有`SELECT`、`DELETE`和`UPDATE`权限。

注意

使用`MERGE`表会带来以下安全问题：如果用户可以访问`MyISAM`表*`t`*，那么该用户可以创建一个访问*`t`*的`MERGE`表*`m`*。然而，如果用户对*`t`*的权限随后被撤销，用户仍然可以通过*`m`*继续访问*`t`*。

使用`DROP TABLE`与`MERGE`表仅删除`MERGE`规范。底层表不受影响。

要创建`MERGE`表，必须指定一个`UNION=(*`table 列表`*)`选项，指示要使用哪些`MyISAM`表。您还可以选择指定`INSERT_METHOD`选项来控制插入到`MERGE`表的方式。使用`FIRST`或`LAST`的值会导致插入到第一个或最后一个底层表中。如果不指定`INSERT_METHOD`选项或指定为`NO`，则不允许向`MERGE`表插入数据，尝试这样做会导致错误。

以下示例展示了如何创建`MERGE`表：

```sql
mysql> CREATE TABLE t1 (
 ->    a INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
 ->    message CHAR(20)) ENGINE=MyISAM;
mysql> CREATE TABLE t2 (
 ->    a INT NOT NULL AUTO_INCREMENT PRIMARY KEY,
 ->    message CHAR(20)) ENGINE=MyISAM;
mysql> INSERT INTO t1 (message) VALUES ('Testing'),('table'),('t1');
mysql> INSERT INTO t2 (message) VALUES ('Testing'),('table'),('t2');
mysql> CREATE TABLE total (
 ->    a INT NOT NULL AUTO_INCREMENT,
 ->    message CHAR(20), INDEX(a))
 ->    ENGINE=MERGE UNION=(t1,t2) INSERT_METHOD=LAST;
```

列`a`在底层`MyISAM`表中作为`PRIMARY KEY`进行索引，但在`MERGE`表中不是。在那里进行了索引，但不是作为`PRIMARY KEY`，因为`MERGE`表无法强制底层表的唯一性。（类似地，在底层表中具有`UNIQUE`索引的列应该在`MERGE`表中进行索引，但不作为`UNIQUE`索引。）

创建`MERGE`表后，可以使用它来发出对整个表组的查询：

```sql
mysql> SELECT * FROM total;
+---+---------+
| a | message |
+---+---------+
| 1 | Testing |
| 2 | table   |
| 3 | t1      |
| 1 | Testing |
| 2 | table   |
| 3 | t2      |
+---+---------+
```

要将`MERGE`表重新映射到不同的`MyISAM`表集合，可以使用以下方法之一：

+   `DROP`掉`MERGE`表并重新创建它。

+   使用`ALTER TABLE *`tbl_name`* UNION=(...)`来更改底层表的列表。

    也可以使用`ALTER TABLE ... UNION=()`（即，使用空的`UNION`子句）来移除所有底层表。然而，在这种情况下，表实际上是空的，插入操作会失败，因为没有底层表来接收新行。这样的表可能用作创建新`MERGE`表的模板，使用`CREATE TABLE ... LIKE`。

底层表的定义和索引必须与`MERGE`表的定义密切符合。符合性是在打开作为`MERGE`表一部分的表时进行检查的，而不是在创建`MERGE`表时进行检查的。如果任何表未通过符合性检查，触发打开表的操作将失败。这意味着更改`MERGE`中表的定义可能会导致在访问`MERGE`表时失败。应用于每个表的符合性检查包括：

+   底层表和`MERGE`表必须具有相同数量的列。

+   底层表和`MERGE`表中的列顺序必须匹配。

+   此外，父`MERGE`表和底层表中每个对应列的规范都会进行比较，并且必须满足这些检查：

    +   底层表和`MERGE`表中的列类型必须相等。

    +   底层表和`MERGE`表中的列长度必须相等。

    +   底层表和`MERGE`表中的列可以是`NULL`。

+   基础表必须至少有与`MERGE`表相同数量的索引。基础表可能比`MERGE`表拥有更多索引，但不能少于`MERGE`表。

    注意

    已知存在一个问题，即相同列上的索引在`MERGE`表和基础`MyISAM`表中必须按相同顺序排列。请参阅 Bug #33653。

    每个索引必须满足这些检查：

    +   基础表和`MERGE`表的索引类型必须相同。

    +   索引定义中的索引部分数量（即，在复合索引内的多个列）必须相同，对应基础表和`MERGE`表。

    +   对于每个索引部分：

        +   索引部分长度必须相等。

        +   索引部分类型必须相等。

        +   索引部分语言必须相等。

        +   检查索引部分是否可以为`NULL`。

如果由于基础表的问题而无法打开或使用`MERGE`表，则`CHECK TABLE`会显示导致问题的表的信息。

### 其他资源

+   专门讨论`MERGE`存储引擎的论坛位于[`forums.mysql.com/list.php?93`](https://forums.mysql.com/list.php?93)。
