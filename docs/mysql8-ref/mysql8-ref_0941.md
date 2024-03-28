# 15.2.5 HANDLER 语句

> 原文：[`dev.mysql.com/doc/refman/8.0/en/handler.html`](https://dev.mysql.com/doc/refman/8.0/en/handler.html)

```sql
HANDLER *tbl_name* OPEN [ [AS] *alias*]

HANDLER *tbl_name* READ *index_name* { = | <= | >= | < | > } (*value1*,*value2*,...)
    [ WHERE *where_condition* ] [LIMIT ... ]
HANDLER *tbl_name* READ *index_name* { FIRST | NEXT | PREV | LAST }
    [ WHERE *where_condition* ] [LIMIT ... ]
HANDLER *tbl_name* READ { FIRST | NEXT }
    [ WHERE *where_condition* ] [LIMIT ... ]

HANDLER *tbl_name* CLOSE
```

`HANDLER`语句提供对表存储引擎接口的直接访问。适用于`InnoDB`和`MyISAM`表。

`HANDLER ... OPEN`语句打开一个表，使其可以通过后续的`HANDLER ... READ`语句访问。此表对象不会被其他会话共享，并且直到会话调用`HANDLER ... CLOSE`或会话终止时才会关闭。

如果使用别名打开表，则必须使用别名而不是表名来引用其他`HANDLER`语句中打开的表。如果不使用别名，而是使用由数据库名限定的表名打开表，则进一步引用必须使用未限定的表名。例如，对于使用`mydb.mytable`打开的表，进一步引用必须使用`mytable`。

第一个`HANDLER ... READ`语法获取一个满足给定值和`WHERE`条件的索引的行。如果有多列索引，请将索引列值指定为逗号分隔的列表。要么为索引中的所有列指定值，要么为索引列的最左前缀指定值。假设索引`my_idx`按顺序包括三列名为`col_a`、`col_b`和`col_c`。`HANDLER`语句可以为索引中的所有三列或最左前缀的列指定值。例如：

```sql
HANDLER ... READ my_idx = (col_a_val,col_b_val,col_c_val) ...
HANDLER ... READ my_idx = (col_a_val,col_b_val) ...
HANDLER ... READ my_idx = (col_a_val) ...
```

要使用`HANDLER`接口引用表的`PRIMARY KEY`，请使用引号标识符``PRIMARY``：

```sql
HANDLER *tbl_name* READ `PRIMARY` ...
```

第二个`HANDLER ... READ`语法按照索引顺序从表中获取与`WHERE`条件匹配的行。

第三个`HANDLER ... READ`语法按照自然行顺序从表中获取与`WHERE`条件匹配的行。当需要进行全表扫描时，比`HANDLER *`tbl_name`* READ *`index_name`*`更快。自然行顺序是`MyISAM`表数据文件中存储行的顺序。此语句也适用于`InnoDB`表，但没有这样的概念，因为没有单独的数据文件。

没有`LIMIT`子句，所有形式的`HANDLER ... READ`如果有可用行，则获取一行。要返回特定数量的行，请包含`LIMIT`子句。其语法与`SELECT`语句相同。参见第 15.2.13 节，“SELECT Statement”。

`HANDLER ... CLOSE`关闭使用`HANDLER ... OPEN`打开的表。

使用`HANDLER`接口而不是普通的`SELECT`语句有几个原因：

+   `HANDLER`比`SELECT`更快：

    +   为`HANDLER ... OPEN`分配一个指定的存储引擎处理程序对象。该对象将用于该表的后续`HANDLER`语句；不需要为每个语句重新初始化。

    +   解析工作较少。

    +   没有优化器或查询检查开销。

    +   处理程序接口不必提供数据的一致外观（例如，允许脏读取），因此存储引擎可以使用`SELECT`通常不允许的优化。

+   `HANDLER`使得将 MySQL 应用程序移植到使用低级`ISAM`-like 接口的应用程序更容易。（参见第 17.20 节，“InnoDB memcached 插件”，了解适应使用键值存储范式的应用程序的另一种方法。）

+   `HANDLER`使您能够以一种难以（甚至不可能）用`SELECT`实现的方式遍历数据库。`HANDLER`接口是在处理提供交互式用户界面到数据库的应用程序时查看数据的更自然方式。

`HANDLER`是一种较低级别的语句。例如，它不提供一致性。也就是说，`HANDLER ... OPEN`不会对表进行快照，并且不会锁定表。这意味着在发出`HANDLER ... OPEN`语句后，表数据可以被修改（由当前会话或其他会话），并且这些修改可能只对`HANDLER ... NEXT`或`HANDLER ... PREV`扫描部分可见。

可以关闭并标记为重新打开打开的处理程序，此时处理程序将失去在表中的位置。当以下两种情况同时成立时会发生这种情况：

+   任何会话在处理程序的表上执行`FLUSH TABLES`或 DDL 语句。

+   打开处理程序的会话执行使用表的非`HANDLER`语句。

对于一个表的`TRUNCATE TABLE`会关闭使用`HANDLER OPEN`打开的表的所有处理程序。

如果使用`FLUSH TABLES *`tbl_name`* WITH READ LOCK`刷新表时，该表是用`HANDLER`打开的，那么处理程序会被隐式刷新并且失去位置。
