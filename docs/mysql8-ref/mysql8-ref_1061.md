> 原文：[`dev.mysql.com/doc/refman/8.0/en/checksum-table.html`](https://dev.mysql.com/doc/refman/8.0/en/checksum-table.html)

#### 15.7.3.3 CHECKSUM TABLE 语句

```sql
CHECKSUM TABLE *tbl_name* [, *tbl_name*] ... [QUICK | EXTENDED]
```

`CHECKSUM TABLE`报告表内容的校验值。您可以使用此语句在备份、回滚或其他旨在将数据恢复到已知状态的操作之前后验证内容是否相同。

这个语句需要表的`SELECT`权限。

这个语句不支持对视图的操作。如果你对视图运行`CHECKSUM TABLE`，`Checksum`值始终为`NULL`，并返回一个警告。

对于不存在的表，`CHECKSUM TABLE`返回`NULL`并生成一个警告。

在校验操作期间，对于`InnoDB`和`MyISAM`，表会被读锁定。

##### 性能考虑

默认情况下，整个表会逐行读取并计算校验值。对于大表，这可能需要很长时间，因此您只会偶尔执行此操作。这种逐行计算是使用`EXTENDED`子句、`InnoDB`和除了`MyISAM`之外的所有其他存储引擎，以及未使用`CHECKSUM=1`子句创建的`MyISAM`表所得到的。

对于使用`CHECKSUM=1`子句创建的`MyISAM`表，`CHECKSUM TABLE`或`CHECKSUM TABLE ... QUICK`返回可以非常快速返回的“实时”表校验值。如果表不符合所有这些条件，`QUICK`方法返回`NULL`。`QUICK`方法不支持`InnoDB`表。有关`CHECKSUM`子句的语法，请参见第 15.1.20 节，“CREATE TABLE Statement”。

校验值取决于表行格式。如果行格式发生变化，校验值也会发生变化。例如，MySQL 5.6 之前的 MySQL 5.6.5 对于诸如`TIME`、`DATETIME`和`TIMESTAMP`等时间类型的存储格式发生了变化，因此如果将一个 5.5 表升级到 MySQL 5.6，校验值可能会发生变化。

重要提示

如果两个表的校验值不同，那么这两个表在某种程度上肯定是不同的。然而，由于`CHECKSUM TABLE`使用的哈希函数不能保证无碰撞，所以两个不完全相同的表可能产生相同的校验值的几率很小。
