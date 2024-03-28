# 17.12.6 通过在线 DDL 简化 DDL 语句

> 译文：[`dev.mysql.com/doc/refman/8.0/en/innodb-online-ddl-single-multi.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-online-ddl-single-multi.html)

在引入在线 DDL 之前，将许多 DDL 操作合并为单个`ALTER TABLE`语句是常见做法。因为每个`ALTER TABLE`语句都涉及复制和重建表，所以一次对同一表进行多个更改更有效，因为这些更改可以在一次表的重建操作中完成。不利之处在于，涉及 DDL 操作的 SQL 代码更难维护和在不同脚本中重复使用。如果每次具体更改都不同，你可能需要为每个略有不同的情况构建一个新的复杂`ALTER TABLE`。

对于可以在线执行的 DDL 操作，你可以将它们分开为单独的`ALTER TABLE`语句，以便更轻松地编写脚本和维护，而不会牺牲效率。例如，你可以将一个复杂的语句简化为：

```sql
ALTER TABLE t1 ADD INDEX i1(c1), ADD UNIQUE INDEX i2(c2),
  CHANGE c4_old_name c4_new_name INTEGER UNSIGNED;
```

并将其分解为可以独立测试和执行的更简单的部分，例如：

```sql
ALTER TABLE t1 ADD INDEX i1(c1);
ALTER TABLE t1 ADD UNIQUE INDEX i2(c2);
ALTER TABLE t1 CHANGE c4_old_name c4_new_name INTEGER UNSIGNED NOT NULL;
```

你可能仍然使用多部分`ALTER TABLE`语句来：

+   必须按特定顺序执行的操作，例如创建索引，然后创建使用该索引的外键约束。

+   所有使用相同特定`LOCK`子句的操作，你希望它们作为一个组要么成功要么失败。

+   无法在线执行的操作，即仍使用表复制方法的操作。

+   为其指定`ALGORITHM=COPY`或`old_alter_table=1`的操作，以在特定情况下需要精确向后兼容性时强制执行表复制行为。
