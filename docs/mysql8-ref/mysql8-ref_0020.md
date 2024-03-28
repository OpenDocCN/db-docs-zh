> 原文：[`dev.mysql.com/doc/refman/8.0/en/constraint-foreign-key.html`](https://dev.mysql.com/doc/refman/8.0/en/constraint-foreign-key.html)

#### 1.6.3.2 外键约束

外键允许您在表之间交叉引用相关数据，而外键约束有助于保持这些分散的数据一致。

MySQL 支持在`CREATE TABLE`和`ALTER TABLE`语句中的`ON UPDATE`和`ON DELETE`外键引用。可用的引用操作包括`RESTRICT`、`CASCADE`、`SET NULL`和`NO ACTION`（默认）。

MySQL 服务器也支持`SET DEFAULT`，但当前被`InnoDB`拒绝为无效。由于 MySQL 不支持延迟约束检查，`NO ACTION`被视为`RESTRICT`。有关 MySQL 支持的外键的确切语法，请参见 Section 15.1.20.5, “FOREIGN KEY Constraints”。

`MATCH FULL`、`MATCH PARTIAL`和`MATCH SIMPLE`是允许的，但应避免使用，因为它们会导致 MySQL 服务器忽略同一语句中使用的任何`ON DELETE`或`ON UPDATE`子句。`MATCH`选项在 MySQL 中没有其他效果，实际上全天候强制执行`MATCH SIMPLE`语义。

MySQL 要求外键列被索引；如果您创建了一个具有外键约束但在给定列上没有索引的表，将会创建一个索引。

你可以从信息模式`KEY_COLUMN_USAGE`表中获取有关外键的信息。这里展示了针对该表的查询示例：

```sql
mysql> SELECT TABLE_SCHEMA, TABLE_NAME, COLUMN_NAME, CONSTRAINT_NAME
     > FROM INFORMATION_SCHEMA.KEY_COLUMN_USAGE
     > WHERE REFERENCED_TABLE_SCHEMA IS NOT NULL;
+--------------+---------------+-------------+-----------------+
| TABLE_SCHEMA | TABLE_NAME    | COLUMN_NAME | CONSTRAINT_NAME |
+--------------+---------------+-------------+-----------------+
| fk1          | myuser        | myuser_id   | f               |
| fk1          | product_order | customer_id | f2              |
| fk1          | product_order | product_id  | f1              |
+--------------+---------------+-------------+-----------------+
3 rows in set (0.01 sec)
```

有关`InnoDB`表上外键的信息也可以在`INFORMATION_SCHEMA`数据库中的`INNODB_FOREIGN`和`INNODB_FOREIGN_COLS`表中找到。

`InnoDB`和`NDB`表支持外键。
