> 原文：[`dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html`](https://dev.mysql.com/doc/refman/8.0/en/create-table-foreign-keys.html)

#### 15.1.20.5 外键约束

MySQL 支持外键，允许在表之间进行相关数据的交叉引用，并支持外键约束，有助于保持相关数据的一致性。

外键关系涉及一个包含初始列值的父表，以及一个包含引用父列值的列值的子表。外键约束定义在子表上。

在`CREATE TABLE`或`ALTER TABLE`语句中定义外键约束的基本语法包括以下内容：

```sql
[CONSTRAINT [*symbol*]] FOREIGN KEY
    [*index_name*] (*col_name*, ...)
    REFERENCES *tbl_name* (*col_name*,...)
    [ON DELETE *reference_option*]
    [ON UPDATE *reference_option*]

*reference_option*:
    RESTRICT | CASCADE | SET NULL | NO ACTION | SET DEFAULT
```

外键约束的使用方法在本节的以下主题中描述：

+   标识符

+   条件和限制

+   参考动作

+   外键约束示例

+   添加外键约束

+   删除外键约束

+   外键检查

+   锁定

+   外键定义和元数据

+   外键错误

##### 标识符

外键约束命名受以下规则约束：

+   如果未定义`CONSTRAINT` *`symbol`*子句，或在`CONSTRAINT`关键字后未包含符号，则会自动生成约束名。

+   如果未定义`CONSTRAINT` *`symbol`*子句，或在`CONSTRAINT`关键字后未包含符号，则会自动生成约束名。

    在 MySQL 8.0.16 之前，如果未定义`CONSTRAINT` *`symbol`*子句，或在`CONSTRAINT`关键字后未包含符号，则`InnoDB`和`NDB`存储引擎将使用`FOREIGN_KEY *`index_name`*`（如果已定义）。在 MySQL 8.0.16 及更高版本中，将忽略`FOREIGN_KEY *`index_name`*`。

+   如果定义了`CONSTRAINT *`symbol`*`值，则必须在数据库中是唯一的。重复的*`symbol`*会导致类似以下错误：ERROR 1005 (HY000): 无法创建表'test.fk1'（错误号：121）。

+   NDB 集群使用创建时的相同大小写存储外键名称。在版本 8.0.20 之前，当处理 `SELECT` 和其他 SQL 语句时，`NDB` 将这些语句中的外键名称与存储的名称进行比较，当 `lower_case_table_names` 等于 0 时，以区分大小写的方式存储。在 NDB 8.0.20 及更高版本中，此值不再影响这些比较是如何进行的，它们总是不考虑大小写地进行。 (Bug #30512043)

在 `FOREIGN KEY ... REFERENCES` 子句中的表格和列标识符可以在反引号内引用 (```sql). Alternatively, double quotation marks (`"`) can be used if the `ANSI_QUOTES` SQL mode is enabled. The `lower_case_table_names` system variable setting is also taken into account.

##### Conditions and Restrictions

Foreign key constraints are subject to the following conditions and restrictions:

*   Parent and child tables must use the same storage engine, and they cannot be defined as temporary tables.

*   Creating a foreign key constraint requires the `REFERENCES` privilege on the parent table.

*   Corresponding columns in the foreign key and the referenced key must have similar data types. *The size and sign of fixed precision types such as `INTEGER` - INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT") and `DECIMAL` - DECIMAL, NUMERIC") must be the same*. The length of string types need not be the same. For nonbinary (character) string columns, the character set and collation must be the same.

*   MySQL supports foreign key references between one column and another within a table. (A column cannot have a foreign key reference to itself.) In these cases, a “child table record” refers to a dependent record within the same table.

*   MySQL requires indexes on foreign keys and referenced keys so that foreign key checks can be fast and not require a table scan. In the referencing table, there must be an index where the foreign key columns are listed as the *first* columns in the same order. Such an index is created on the referencing table automatically if it does not exist. This index might be silently dropped later if you create another index that can be used to enforce the foreign key constraint. *`index_name`*, if given, is used as described previously.

*   `InnoDB` permits a foreign key to reference any index column or group of columns. However, in the referenced table, there must be an index where the referenced columns are the *first* columns in the same order. Hidden columns that `InnoDB` adds to an index are also considered (see Section 17.6.2.1, “Clustered and Secondary Indexes”).

    `NDB` requires an explicit unique key (or primary key) on any column referenced as a foreign key. `InnoDB` does not, which is an extension of standard SQL.

*   Index prefixes on foreign key columns are not supported. Consequently, `BLOB` and `TEXT` columns cannot be included in a foreign key because indexes on those columns must always include a prefix length.

*   `InnoDB` does not currently support foreign keys for tables with user-defined partitioning. This includes both parent and child tables.

    This restriction does not apply for `NDB` tables that are partitioned by `KEY` or `LINEAR KEY` (the only user partitioning types supported by the `NDB` storage engine); these may have foreign key references or be the targets of such references.

*   A table in a foreign key relationship cannot be altered to use another storage engine. To change the storage engine, you must drop any foreign key constraints first.

*   A foreign key constraint cannot reference a virtual generated column.

For information about how the MySQL implementation of foreign key constraints differs from the SQL standard, see Section 1.6.2.3, “FOREIGN KEY Constraint Differences”.

##### Referential Actions

When an `UPDATE` or `DELETE` operation affects a key value in the parent table that has matching rows in the child table, the result depends on the *referential action* specified by `ON UPDATE` and `ON DELETE` subclauses of the `FOREIGN KEY` clause. Referential actions include:

*   `CASCADE`: Delete or update the row from the parent table and automatically delete or update the matching rows in the child table. Both `ON DELETE CASCADE` and `ON UPDATE CASCADE` are supported. Between two tables, do not define several `ON UPDATE CASCADE` clauses that act on the same column in the parent table or in the child table.

    If a `FOREIGN KEY` clause is defined on both tables in a foreign key relationship, making both tables a parent and child, an `ON UPDATE CASCADE` or `ON DELETE CASCADE` subclause defined for one `FOREIGN KEY` clause must be defined for the other in order for cascading operations to succeed. If an `ON UPDATE CASCADE` or `ON DELETE CASCADE` subclause is only defined for one `FOREIGN KEY` clause, cascading operations fail with an error.

    Note

    Cascaded foreign key actions do not activate triggers.

*   `SET NULL`: Delete or update the row from the parent table and set the foreign key column or columns in the child table to `NULL`. Both `ON DELETE SET NULL` and `ON UPDATE SET NULL` clauses are supported.

    If you specify a `SET NULL` action, *make sure that you have not declared the columns in the child table as `NOT NULL`*.

*   `RESTRICT`: Rejects the delete or update operation for the parent table. Specifying `RESTRICT` (or `NO ACTION`) is the same as omitting the `ON DELETE` or `ON UPDATE` clause.

*   `NO ACTION`: A keyword from standard SQL. For `InnoDB`, this is equivalent to `RESTRICT`; the delete or update operation for the parent table is immediately rejected if there is a related foreign key value in the referenced table. `NDB` supports deferred checks, and `NO ACTION` specifies a deferred check; when this is used, constraint checks are not performed until commit time. Note that for `NDB` tables, this causes all foreign key checks made for both parent and child tables to be deferred.

*   `SET DEFAULT`: This action is recognized by the MySQL parser, but both `InnoDB` and `NDB` reject table definitions containing `ON DELETE SET DEFAULT` or `ON UPDATE SET DEFAULT` clauses.

For storage engines that support foreign keys, MySQL rejects any `INSERT` or `UPDATE` operation that attempts to create a foreign key value in a child table if there is no matching candidate key value in the parent table.

For an `ON DELETE` or `ON UPDATE` that is not specified, the default action is always `NO ACTION`.

As the default, an `ON DELETE NO ACTION` or `ON UPDATE NO ACTION` clause that is specified explicitly does not appear in `SHOW CREATE TABLE` output or in tables dumped with **mysqldump**. `RESTRICT`, which is an equivalent non-default keyword, appears in `SHOW CREATE TABLE` output and in tables dumped with **mysqldump**.

For `NDB` tables, `ON UPDATE CASCADE` is not supported where the reference is to the parent table's primary key.

As of NDB 8.0.16: For `NDB` tables, `ON DELETE CASCADE` is not supported where the child table contains one or more columns of any of the `TEXT` or `BLOB` types. (Bug #89511, Bug #27484882)

`InnoDB` performs cascading operations using a depth-first search algorithm on the records of the index that corresponds to the foreign key constraint.

A foreign key constraint on a stored generated column cannot use `CASCADE`, `SET NULL`, or `SET DEFAULT` as `ON UPDATE` referential actions, nor can it use `SET NULL` or `SET DEFAULT` as `ON DELETE` referential actions.

A foreign key constraint on the base column of a stored generated column cannot use `CASCADE`, `SET NULL`, or `SET DEFAULT` as `ON UPDATE` or `ON DELETE` referential actions.

##### Foreign Key Constraint Examples

This simple example relates `parent` and `child` tables through a single-column foreign key:

```

创建表格父 (

    id INT 不为空,

    主键 (id)

) 引擎=INNODB;

创建表格子表 (

    id INT,

    parent_id INT,

    索引 par_ind (parent_id),

    外键 (parent_id)

        REFERENCES 父(id)

        在删除时级联

) 引擎=INNODB;

```sql

This is a more complex example in which a `product_order` table has foreign keys for two other tables. One foreign key references a two-column index in the `product` table. The other references a single-column index in the `customer` table:

```

创建表格产品 (

    category INT NOT NULL, id INT NOT NULL,

    价格 DECIMAL,

    主键(category, id)

)   引擎=INNODB;

创建表格顾客 (

    id INT 不为空,

    主键 (id)

)   引擎=INNODB;

创建表格产品订单 (

    no INT 不为空 自动增量,

    产品类别 INT 不为空,

    product_id INT NOT NULL,

    顾客 id INT 不为空,

    主键(no),

    索引 (产品类别, 产品 id),

    索引 (顾客 id),

    外键 (产品类别, 产品 id)

    REFERENCES 产品(类别, id)

    在更新时级联在删除时限制,

    外键 (customer_id)

    REFERENCES 顾客(id)

)   引擎=INNODB;

```sql

##### Adding Foreign Key Constraints

You can add a foreign key constraint to an existing table using the following `ALTER TABLE` syntax:

```

ALTER TABLE *tbl_name*

    添加 [约束 [*symbol*]] 外键

    [*index_name*] (*col_name*, ...)

    REFERENCES *tbl_name* (*col_name*,...)

    [ON DELETE *reference_option*]

    [在更新 *reference_option*]

```sql

The foreign key can be self referential (referring to the same table). When you add a foreign key constraint to a table using `ALTER TABLE`, *remember to first create an index on the column(s) referenced by the foreign key.*

##### Dropping Foreign Key Constraints

You can drop a foreign key constraint using the following `ALTER TABLE` syntax:

```

ALTER TABLE *tbl_name* DROP FOREIGN KEY *fk_symbol*;

```sql

If the `FOREIGN KEY` clause defined a `CONSTRAINT` name when you created the constraint, you can refer to that name to drop the foreign key constraint. Otherwise, a constraint name was generated internally, and you must use that value. To determine the foreign key constraint name, use `SHOW CREATE TABLE`:

```

mysql> SHOW CREATE TABLE child\G

*************************** 1\. 行 ***************************

    表格：子表

创建表格：CREATE TABLE `child` (

`id` int DEFAULT NULL,

`parent_id` int DEFAULT NULL,

键 `par_ind` (`parent_id`),

CONSTRAINT `child_ibfk_1` FOREIGN KEY (`parent_id`)

REFERENCES `parent` (`id`) ON DELETE CASCADE

) 引擎=InnoDB 默认字符集=utf8mb4 校对=utf8mb4_0900_ai_ci

mysql> ALTER TABLE child DROP FOREIGN KEY `child_ibfk_1`;

```sql

Adding and dropping a foreign key in the same `ALTER TABLE` statement is supported for `ALTER TABLE ... ALGORITHM=INPLACE`. It is not supported for `ALTER TABLE ... ALGORITHM=COPY`.

##### Foreign Key Checks

In MySQL, InnoDB and NDB tables support checking of foreign key constraints. Foreign key checking is controlled by the `foreign_key_checks` variable, which is enabled by default. Typically, you leave this variable enabled during normal operation to enforce referential integrity. The `foreign_key_checks` variable has the same effect on `NDB` tables as it does for `InnoDB` tables.

The `foreign_key_checks` variable is dynamic and supports both global and session scopes. For information about using system variables, see Section 7.1.9, “Using System Variables”.

Disabling foreign key checking is useful when:

*   Dropping a table that is referenced by a foreign key constraint. A referenced table can only be dropped after `foreign_key_checks` is disabled. When you drop a table, constraints defined on the table are also dropped.

*   Reloading tables in different order than required by their foreign key relationships. For example, **mysqldump** produces correct definitions of tables in the dump file, including foreign key constraints for child tables. To make it easier to reload dump files for tables with foreign key relationships, **mysqldump** automatically includes a statement in the dump output that disables `foreign_key_checks`. This enables you to import the tables in any order in case the dump file contains tables that are not correctly ordered for foreign keys. Disabling `foreign_key_checks` also speeds up the import operation by avoiding foreign key checks.

*   Executing `LOAD DATA` operations, to avoid foreign key checking.

*   Performing an `ALTER TABLE` operation on a table that has a foreign key relationship.

When `foreign_key_checks` is disabled, foreign key constraints are ignored, with the following exceptions:

*   Recreating a table that was previously dropped returns an error if the table definition does not conform to the foreign key constraints that reference the table. The table must have the correct column names and types. It must also have indexes on the referenced keys. If these requirements are not satisfied, MySQL returns Error 1005 that refers to errno: 150 in the error message, which means that a foreign key constraint was not correctly formed.

*   Altering a table returns an error (errno: 150) if a foreign key definition is incorrectly formed for the altered table.

*   Dropping an index required by a foreign key constraint. The foreign key constraint must be removed before dropping the index.

*   Creating a foreign key constraint where a column references a nonmatching column type.

Disabling `foreign_key_checks` has these additional implications:

*   It is permitted to drop a database that contains tables with foreign keys that are referenced by tables outside the database.

*   It is permitted to drop a table with foreign keys referenced by other tables.

*   Enabling `foreign_key_checks` does not trigger a scan of table data, which means that rows added to a table while `foreign_key_checks` is disabled are not checked for consistency when `foreign_key_checks` is re-enabled.

##### Locking

MySQL extends metadata locks, as necessary, to tables that are related by a foreign key constraint. Extending metadata locks prevents conflicting DML and DDL operations from executing concurrently on related tables. This feature also enables updates to foreign key metadata when a parent table is modified. In earlier MySQL releases, foreign key metadata, which is owned by the child table, could not be updated safely.

If a table is locked explicitly with `LOCK TABLES`, any tables related by a foreign key constraint are opened and locked implicitly. For foreign key checks, a shared read-only lock (`LOCK TABLES READ`) is taken on related tables. For cascading updates, a shared-nothing write lock (`LOCK TABLES WRITE`) is taken on related tables that are involved in the operation.

##### Foreign Key Definitions and Metadata

To view a foreign key definition, use `SHOW CREATE TABLE`:

```

mysql> SHOW CREATE TABLE child\G

*************************** 1\. 行 ***************************

    表格：子表

创建表格：CREATE TABLE `child` (

`id` int DEFAULT NULL,

`parent_id` int DEFAULT NULL,

键 `par_ind` (`parent_id`),

CONSTRAINT `child_ibfk_1` FOREIGN KEY (`parent_id`)

REFERENCES `parent` (`id`) ON DELETE CASCADE

) 引擎=InnoDB 默认字符集=utf8mb4 校对=utf8mb4_0900_ai_ci

```sql

You can obtain information about foreign keys from the Information Schema `KEY_COLUMN_USAGE` table. An example of a query against this table is shown here:

```

mysql> 选择 表格模式, 表格名称, 列名, 约束名称

    来自 INFORMATION_SCHEMA.KEY_COLUMN_USAGE

    当引用表格模式不为空时;

+--------------+------------+-------------+-----------------+

| 表格模式 | 表格名称 | 列名 | 约束名称 |
| --- | --- | --- | --- |

+--------------+------------+-------------+-----------------+

| test         | child      | parent_id   | child_ibfk_1    |
| --- | --- | --- | --- |

+--------------+------------+-------------+-----------------+

```sql

You can obtain information specific to `InnoDB` foreign keys from the `INNODB_FOREIGN` and `INNODB_FOREIGN_COLS` tables. Example queries are show here:

```

mysql> 从 INFORMATION_SCHEMA.INNODB_FOREIGN 表中选择 * \G

*************************** 1\. 行 ***************************

    ID: test/child_ibfk_1

FOR_NAME: test/child

REF_NAME: test/parent

N_COLS: 1

    类型: 1

mysql> 从 INFORMATION_SCHEMA.INNODB_FOREIGN_COLS 表中选择 * \G

*************************** 1\. 行 ***************************

        ID: test/child_ibfk_1

FOR_COL_NAME: parent_id

REF_COL_NAME: id

        位置: 0

```sql

##### Foreign Key Errors

In the event of a foreign key error involving `InnoDB` tables (usually Error 150 in the MySQL Server), information about the latest foreign key error can be obtained by checking `SHOW ENGINE INNODB STATUS` output.

```

mysql> 显示引擎 INNODB 状态\G

...

------------------------

最新的外键错误

------------------------

2018-04-12 14:57:24 0x7f97a9c91700 事务:

事务 7717，活跃 0 秒插入

使用中的 mysql 表 1，已锁定 1

4 个锁结构，堆大小 1136，3 个行锁，撤销日志条目 3

MySQL 线程 id 8，OS 线程句柄 140289365317376，查询 id 14 localhost root 更新

插入到 child 表中的值 (NULL, 1), (NULL, 2), (NULL, 3), (NULL, 4), (NULL, 5), (NULL, 6)

对于表 `test`.`child` 的外键约束失败：

,

约束 `child_ibfk_1` 外键 (`parent_id`) 引用 `parent` 表 (`id`) 在删除时

级联更新级联

尝试在子表中添加，索引 par_ind 元组：

数据元组: 2 个字段;

0: 长度 4; 十六进制 80000003; ASCII     ;;

1: 长度 4; 十六进制 80000003; ASCII     ;;

但在父表 `test`.`parent` 中，在主键 PRIMARY 中，

我们能找到的最接近的匹配是记录：

物理记录: 字段数 3; 紧凑格式; 信息位 0

0: 长度 4; 十六进制 80000004; ASCII     ;;

1: 长度 6; 十六进制 000000001e19; ASCII       ;;

2: 长度 7; 十六进制 81000001110137; ASCII       7;;

...

```

警告

如果用户对所有父表具有表级别权限，则外键操作的 `ER_NO_REFERENCED_ROW_2` 和 `ER_ROW_IS_REFERENCED_2` 错误消息会暴露有关父表的信息。如果用户对所有父表没有表级别权限，则显示更通用的错误消息 (`ER_NO_REFERENCED_ROW` 和 `ER_ROW_IS_REFERENCED`)。

一个例外是，对于定义为使用 `DEFINER` 权限执行的存储程序，权限评估的用户是程序中 `DEFINER` 子句中的用户，而不是调用用户。如果该用户具有表级别父表权限，父表信息仍然会显示。在这种情况下，存储程序创建者有责任通过包含适当的条件处理程序来隐藏信息。
