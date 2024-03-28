# 10.6.1 优化 MyISAM 查询

> 原文：[`dev.mysql.com/doc/refman/8.0/en/optimizing-queries-myisam.html`](https://dev.mysql.com/doc/refman/8.0/en/optimizing-queries-myisam.html)

一些加快`MyISAM`表查询速度的一般提示：

+   为了帮助 MySQL 更好地优化查询，请使用`ANALYZE TABLE`或在加载数据后对表运行**myisamchk --analyze**。 这将更新每个索引部分的一个值，该值表示具有相同值的平均行数。 （对于唯一索引，这始终是 1。） MySQL 在基于非常量表达式连接两个表时使用此值来决定选择哪个索引。 您可以通过使用`SHOW INDEX FROM *`tbl_name`*`并检查`Cardinality`值来检查表分析的结果。 **myisamchk --description --verbose**显示索引分布信息。

+   要根据索引对索引和数据进行排序，请使用**myisamchk --sort-index --sort-records=1**（假设您要按索引 1 排序）。 如果您有一个想要按照索引顺序读取所有行的唯一索引，这是使查询更快的好方法。 第一次以这种方式对大表进行排序可能需要很长时间。

+   尽量避免在频繁更新的`MyISAM`表上执行复杂的`SELECT`查询，以避免由于读者和写者之间的争用而导致的表锁定问题。

+   `MyISAM`支持并发插入：如果表在数据文件中间没有空闲块，则可以在其他线程从表中读取时向其中插入新行。 如果能够这样做很重要，请考虑以避免删除行的方式使用表。 另一种可能性是在从表中删除大量行后运行`OPTIMIZE TABLE`来碎片整理表。 通过设置`concurrent_insert`变量来改变此行为。 您可以强制追加新行（从而允许并发插入），即使在已删除行的表中也是如此。 请参阅第 10.11.3 节，“并发插入”。

+   对于频繁更改的`MyISAM`表，请尽量避免所有可变长度列（`VARCHAR`、`BLOB`和`TEXT`）。如果表包含任何一个可变长度列，表将使用动态行格式。参见 Chapter 18, *Alternative Storage Engines*。

+   通常不值得将表拆分为不同的表，只是因为行变得很大。在访问行时，最大的性能损失是需要查找行的第一个字节的磁盘寻道。找到数据后，大多数现代磁盘可以以足够快的速度读取整个行，适用于大多数应用程序。唯一需要拆分表的情况是，如果是使用动态行格式的`MyISAM`表，您可以将其更改为固定行大小，或者如果您经常需要扫描表但不需要大多数列。参见 Chapter 18, *Alternative Storage Engines*。

+   如果您通常按`*`expr1`*, *`expr2`*, ...`顺序检索行，则使用`ALTER TABLE ... ORDER BY *`expr1`*, *`expr2`*, ...`。在对表进行广泛更改后使用此选项，您可能能够获得更高的性能。

+   如果您经常需要根据大量行的信息计算结果，可能最好引入一个新表并实时更新计数器。以下形式的更新非常快：

    ```sql
    UPDATE *tbl_name* SET *count_col*=*count_col*+1 WHERE *key_col*=*constant*;
    ```

    当您使用只有表级锁定（多个读者与单个写者）的 MySQL 存储引擎，如`MyISAM`时，这一点非常重要。这也可以提高大多数数据库系统的性能，因为在这种情况下，行锁定管理器的工作量较少。

+   定期使用`OPTIMIZE TABLE`以避免动态格式`MyISAM`表的碎片化。参见 Section 18.2.3, “MyISAM Table Storage Formats”。

+   使用`DELAY_KEY_WRITE=1`表选项声明`MyISAM`表可以使索引更新更快，因为它们在表关闭之前不会刷新到磁盘。缺点是，如果在打开这样的表时有什么东西终止了服务器，您必须通过设置`myisam_recover_options`系统变量来确保表的正常，或者在重新启动服务器之前运行**myisamchk**。 （然而，即使在这种情况下，使用`DELAY_KEY_WRITE`也不会丢失任何内容，因为关键信息始终可以从数据行生成。）

+   在`MyISAM`索引中，字符串会自动进行前缀和末尾空格压缩。参见第 15.1.15 节，“CREATE INDEX 语句”。

+   通过在应用程序中缓存查询或答案，然后一次性执行多个插入或更新操作，可以提高性能。在此操作期间锁定表确保索引缓存仅在所有更新完成后刷新一次。
