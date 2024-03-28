# 28.1 介绍

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-introduction.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-introduction.html)

`INFORMATION_SCHEMA`提供对数据库元数据的访问，包括有关 MySQL 服务器的信息，如数据库或表的名称、列的数据类型或访问权限。有时用于表示此信息的其他术语是数据字典和系统目录。

+   INFORMATION_SCHEMA 使用注意事项

+   字符集考虑

+   INFORMATION_SCHEMA 作为 SHOW 语句的替代

+   INFORMATION_SCHEMA 和权限

+   性能考虑

+   标准考虑

+   INFORMATION_SCHEMA 参考部分中的约定

+   相关信息

### INFORMATION_SCHEMA 使用注意事项

`INFORMATION_SCHEMA`是每个 MySQL 实例中的一个数据库，存储有关 MySQL 服务器维护的所有其他数据库的信息的地方。`INFORMATION_SCHEMA`数据库包含几个只读表。它们实际上是视图，而不是基本表，因此与它们关联的文件不存在，您不能在它们上设置触发器。此外，没有以该名称命名的数据库目录。

尽管您可以使用`USE`语句将`INFORMATION_SCHEMA`选择为默认数据库，但您只能读取表的内容，而不能对其执行`INSERT`、`UPDATE`或`DELETE`操作。

这里是一个从`INFORMATION_SCHEMA`中检索信息的语句示例：

```sql
mysql> SELECT table_name, table_type, engine
       FROM information_schema.tables
       WHERE table_schema = 'db5'
       ORDER BY table_name;
+------------+------------+--------+
| table_name | table_type | engine |
+------------+------------+--------+
| fk         | BASE TABLE | InnoDB |
| fk2        | BASE TABLE | InnoDB |
| goto       | BASE TABLE | MyISAM |
| into       | BASE TABLE | MyISAM |
| k          | BASE TABLE | MyISAM |
| kurs       | BASE TABLE | MyISAM |
| loop       | BASE TABLE | MyISAM |
| pk         | BASE TABLE | InnoDB |
| t          | BASE TABLE | MyISAM |
| t2         | BASE TABLE | MyISAM |
| t3         | BASE TABLE | MyISAM |
| t7         | BASE TABLE | MyISAM |
| tables     | BASE TABLE | MyISAM |
| v          | VIEW       | NULL   |
| v2         | VIEW       | NULL   |
| v3         | VIEW       | NULL   |
| v56        | VIEW       | NULL   |
+------------+------------+--------+
17 rows in set (0.01 sec)
```

解释：该语句请求列出数据库`db5`中所有表的列表，仅显示三个信息：表的名称、类型和存储引擎。

从 MySQL 8.0.30 开始，默认情况下，描述表列、键或两者的所有`INFORMATION_SCHEMA`表中可见生成的不可见主键的信息，例如`COLUMNS`和`STATISTICS`表。如果您希望使这些信息对从这些表中选择的查询隐藏，可以通过将`show_gipk_in_create_table_and_information_schema`服务器系统变量的值设置为`OFF`来实现。有关更多信息，请参见 Section 15.1.20.11，“生成的不可见主键”。

### 字符集考虑

字符列的定义（例如，`TABLES.TABLE_NAME`）通常是`VARCHAR(*`N`*) CHARACTER SET utf8mb3`，其中*`N`*至少为 64。MySQL 对此字符集（`utf8mb3_general_ci`）使用默认排序规则进行所有搜索、排序、比较和其他字符串操作。

因为一些 MySQL 对象表示为文件，对`INFORMATION_SCHEMA`字符串列的搜索可能会受到文件系统的大小写敏感性的影响。有关更多信息，请参见 Section 12.8.7，“在 INFORMATION_SCHEMA 搜索中使用排序规则”。

### INFORMATION_SCHEMA 作为 SHOW 语句的替代

`SELECT ... FROM INFORMATION_SCHEMA`语句旨在提供一种更一致的方式来访问 MySQL 支持的各种`SHOW`语句提供的信息（`SHOW DATABASES`、`SHOW TABLES`等）。与`SHOW`相比，使用`SELECT`具有以下优势：

+   它符合 Codd 的规则，因为所有访问都是在表上进行的。

+   您可以使用`SELECT`语句的熟悉语法，只需学习一些表和列名称。

+   实施者无需担心添加关键字。

+   您可以将`INFORMATION_SCHEMA`查询的结果进行过滤、排序、连接和转换为应用程序需要的任何格式，例如数据结构或文本表示以进行解析。

+   这种技术与其他数据库系统更具互操作性。例如，Oracle Database 用户熟悉在 Oracle 数据字典中查询表。

由于`SHOW`是熟悉且广泛使用的，`SHOW`语句仍然作为一种选择。实际上，随着`INFORMATION_SCHEMA`的实现，`SHOW`有一些增强，如第 28.8 节，“SHOW 语句的扩展”中所述。

### INFORMATION_SCHEMA 和权限

对于大多数`INFORMATION_SCHEMA`表，每个 MySQL 用户都有权访问它们，但只能看到与用户具有适当访问权限的对象对应的表中的行。在某些情况下（例如`INFORMATION_SCHEMA` `ROUTINES`表中的`ROUTINE_DEFINITION`列），权限不足的用户会看到`NULL`。一些表具有不同的权限要求；对于这些表，要求在适用的表描述中提到。例如，`InnoDB`表（以`INNODB_`开头的表）需要`PROCESS`权限。

选择从`INFORMATION_SCHEMA`中获取信息和通过`SHOW`语句查看相同信息的权限是相同的。在任何情况下，您必须对对象具有某些权限才能查看有关它的信息。

### 性能考虑

从多个数据库中搜索信息的`INFORMATION_SCHEMA`查询可能需要很长时间并影响性能。要检查查询的效率，可以使用`EXPLAIN`。有关使用`EXPLAIN`输出来调整`INFORMATION_SCHEMA`查询的信息，请参阅第 10.2.3 节，“优化 INFORMATION_SCHEMA 查询”。

### 标准考虑

MySQL 中`INFORMATION_SCHEMA`表结构的实现遵循 ANSI/ISO SQL:2003 标准第 11 部分*Schemata*。我们的目标是与 SQL:2003 核心功能 F021 *基本信息模式*大致符合。

使用 SQL Server 2000（也遵循标准）的用户可能会注意到很强的相似性。然而，MySQL 省略了许多对我们实现不相关的列，并添加了 MySQL 特定的列。`INFORMATION_SCHEMA` `TABLES`表中的`ENGINE`列就是这样一个添加的列。

尽管其他 DBMS 使用各种名称，如`syscat`或`system`，但标准名称是`INFORMATION_SCHEMA`。

为避免使用标准中保留的任何名称或在 DB2、SQL Server 或 Oracle 中保留的名称，我们更改了一些标记为“MySQL 扩展”的列的名称。（例如，在 `TABLES` 表中，我们将 `COLLATION` 更改为 `TABLE_COLLATION`。）请参阅本文末尾的保留字列表：[`web.archive.org/web/20070428032454/http://www.dbazine.com/db2/db2-disarticles/gulutzan5`](https://web.archive.org/web/20070428032454/http://www.dbazine.com/db2/db2-disarticles/gulutzan5)。

### INFORMATION_SCHEMA 参考部分的约定

以下部分描述了 `INFORMATION_SCHEMA` 中的每个表和列。对于每列，有三个信息：

+   “`INFORMATION_SCHEMA` Name” 表示 `INFORMATION_SCHEMA` 表中列的名称。除非“备注”字段说“MySQL 扩展”，否则这对应于标准 SQL 名称。

+   “`SHOW` Name” 表示最接近的 `SHOW` 语句中的等效字段名称，如果有的话。

+   “备注”在适用时提供额外信息。如果此字段为 `NULL`，则表示列的值始终为 `NULL`。如果此字段说“MySQL 扩展”，则该列是标准 SQL 的 MySQL 扩展。

许多部分指示了 `SHOW` 语句等效于从 `INFORMATION_SCHEMA` 检索信息的 `SELECT` 语句。对于 `SHOW` 语句，如果省略 `FROM *`db_name`*` 子句，则会显示默认数据库的信息，您可以通过在检索信息的查询的 `WHERE` 子句中添加 `AND TABLE_SCHEMA = SCHEMA()` 条件来选择默认数据库的信息。

### 相关信息

这些部分讨论了其他与 `INFORMATION_SCHEMA` 相关的主题：

+   有关 `InnoDB` 存储引擎特定的 `INFORMATION_SCHEMA` 表的信息：第 28.4 节，“INFORMATION_SCHEMA InnoDB 表”

+   有关线程池插件特定的 `INFORMATION_SCHEMA` 表的信息：第 28.5 节，“INFORMATION_SCHEMA 线程池表”

+   有关 `CONNECTION_CONTROL` 插件特定的 `INFORMATION_SCHEMA` 表的信息：第 28.6 节，“INFORMATION_SCHEMA 连接控制表”

+   关于 `INFORMATION_SCHEMA` 数据库经常被问到的问题的答案：第 A.7 节，“MySQL 8.0 FAQ: INFORMATION_SCHEMA”

+   `INFORMATION_SCHEMA` 查询和优化器：第 10.2.3 节，“优化 INFORMATION_SCHEMA 查询”

+   校对对 `INFORMATION_SCHEMA` 比较的影响：第 12.8.7 节，“在 INFORMATION_SCHEMA 搜索中使用校对”
