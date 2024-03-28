# 15.1.33 DROP TABLESPACE Statement

> 原文：[`dev.mysql.com/doc/refman/8.0/en/drop-tablespace.html`](https://dev.mysql.com/doc/refman/8.0/en/drop-tablespace.html)

```sql
DROP [UNDO] TABLESPACE *tablespace_name*
    [ENGINE [=] *engine_name*]
```

此语句删除先前使用`CREATE TABLESPACE`创建的表空间。它受`NDB`和`InnoDB`存储引擎支持。

在 MySQL 8.0.14 中引入的`UNDO`关键字必须在删除撤销表空间时指定。只能删除使用`CREATE UNDO TABLESPACE`语法创建的撤销表空间。撤销表空间必须处于`空`状态才能删除。有关更多信息，请参见 Section 17.6.3.4, “Undo Tablespaces”。

`ENGINE`设置使用表空间的存储引擎，其中*`engine_name`*是存储引擎的名称。目前，支持值为`InnoDB`和`NDB`。如果未设置，则使用`default_storage_engine`的值。如果与用于创建表空间的存储引擎不同，则`DROP TABLESPACE`语句将失败。

`*`tablespace_name`*`是 MySQL 中区分大小写的标识符。

对于`InnoDB`通用表空间，在执行`DROP TABLESPACE`操作之前，必须从表空间中删除所有表。如果表空间不为空，`DROP TABLESPACE`会返回错误。

要删除的`NDB`表空间不能包含任何数据文件；换句话说，在您可以删除`NDB`表空间之前，必须先使用`ALTER TABLESPACE ... DROP DATAFILE`命令删除每个数据文件。

#### 注意

+   当表空间中的最后一个表被删除时，通用`InnoDB`表空间不会自动删除。必须使用`DROP TABLESPACE *`tablespace_name`*`显式删除表空间。

+   `DROP DATABASE`操作可以删除属于通用表空间的表，但无法删除表空间，即使操作删除了属于表空间的所有表。必须使用`DROP TABLESPACE *`tablespace_name`*`显式删除表空间。

+   与系统表空间类似，截断或删除存储在通用表空间中的表会在通用表空间的.ibd 数据文件中创建内部的可用空间，该空间只能用于新的`InnoDB`数据。与针对每个表的文件表空间不同，空间不会像对操作系统一样释放回去。

#### InnoDB 示例

这个示例演示了如何删除一个`InnoDB`通用表空间。通用表空间`ts1`是用一个表创建的。在删除表空间之前，必须先删除表。

```sql
mysql> CREATE TABLESPACE `ts1` ADD DATAFILE 'ts1.ibd' Engine=InnoDB;

mysql> CREATE TABLE t1 (c1 INT PRIMARY KEY) TABLESPACE ts1 Engine=InnoDB;

mysql> DROP TABLE t1;

mysql> DROP TABLESPACE ts1;
```

本示例演示了如何删除一个撤销表空间。撤销表空间必须处于`空`状态才能被删除。有关更多信息，请参见第 17.6.3.4 节，“Undo Tablespaces”。

```sql
mysql> DROP UNDO TABLESPACE *undo_003*;
```

#### NDB 示例

本示例演示了如何在首先创建表空间后删除一个`NDB`表空间`myts`，假设存在一个名为`mylg`的日志文件组（参见第 15.1.16 节，“CREATE LOGFILE GROUP Statement”）。

```sql
mysql> CREATE TABLESPACE myts
 ->     ADD DATAFILE 'mydata-1.dat'
 ->     USE LOGFILE GROUP mylg
 ->     ENGINE=NDB;
```

必须使用`ALTER TABLESPACE`语句，如下所示，从表空间中删除所有数据文件，然后才能删除表空间：

```sql
mysql> ALTER TABLESPACE myts
 ->     DROP DATAFILE 'mydata-1.dat'
 ->     ENGINE=NDB;

mysql> DROP TABLESPACE myts;
```
