# 17.21.3 强制 InnoDB 恢复

> 原文：[`dev.mysql.com/doc/refman/8.0/en/forcing-innodb-recovery.html`](https://dev.mysql.com/doc/refman/8.0/en/forcing-innodb-recovery.html)

为了调查数据库页面损坏，您可以使用 `SELECT ... INTO OUTFILE` 从数据库中导出表。通常，以这种方式获得的大部分数据是完整的。严重的损坏可能会导致 `SELECT * FROM *`tbl_name`*` 语句或 `InnoDB` 后台操作意外退出或断言，甚至导致 `InnoDB` 回滚恢复崩溃。在这种情况下，您可以使用 `innodb_force_recovery` 选项来强制 `InnoDB` 存储引擎启动，同时阻止后台操作运行，以便您可以导出表。例如，您可以在重新启动服务器之前将以下行添加到您的选项文件的 `[mysqld]` 部分中：

```sql
[mysqld]
innodb_force_recovery = 1
```

有关使用选项文件的信息，请参见 Section 6.2.2.2, “Using Option Files”。

警告

仅在紧急情况下将 `innodb_force_recovery` 设置为大于 0 的值，以便您可以启动 `InnoDB` 并导出表。在这样做之前，请确保您有数据库的备份副本，以防需要重新创建它。值为 4 或更大可能会永久损坏数据文件。仅在成功在数据库的单独物理副本上测试设置后，才在生产服务器实例上使用大于 4 的 `innodb_force_recovery` 设置。在强制 `InnoDB` 恢复时，您应始终从 `innodb_force_recovery=1` 开始，并根据需要逐渐增加值。

`innodb_force_recovery` 默认为 0（正常启动，无需强制恢复）。`innodb_force_recovery` 允许的非零值为 1 到 6。较大的值包含较小值的功能。例如，值为 3 包含值为 1 和 2 的所有功能。

如果您能够使用值为 3 或更低的 `innodb_force_recovery` 导出表，那么您相对安全，只有一些损坏的单个页面上的数据丢失。值为 4 或更大被认为是危险的，因为数据文件可能会永久损坏。值为 6 被认为是极端的，因为数据库页面处于过时状态，这反过来可能会在 B 树 和其他数据库结构中引入更多损坏。

作为安全措施，当`innodb_force_recovery`大于 0 时，`InnoDB`会阻止`INSERT`、`UPDATE`或`DELETE`操作。设置为 4 或更高的`innodb_force_recovery`值会将`InnoDB`置于只读模式。

+   `1` (`SRV_FORCE_IGNORE_CORRUPT`)

    让服务器即使检测到损坏的页面也能继续运行。尝试使`SELECT * FROM * tbl_name *`跳过损坏的索引记录和页面，有助于导出表格。

+   `2` (`SRV_FORCE_NO_BACKGROUND`)

    阻止主线程和任何清理线程运行。如果在清理操作期间发生意外退出，此恢复值将阻止它。

+   `3` (`SRV_FORCE_NO_TRX_UNDO`)

    在崩溃恢复后不运行事务回滚。

+   `4` (`SRV_FORCE_NO_IBUF_MERGE`)

    阻止插入缓冲区合并操作。如果这些操作可能导致崩溃，则不执行。不计算表的统计信息。这个值可能会永久损坏数据文件。使用此值后，准备好删除并重新创建所有辅助索引。将`InnoDB`设置为只读。

+   `5` (`SRV_FORCE_NO_UNDO_LOG_SCAN`)

    在启动数据库时不查看撤销日志：`InnoDB`将即使是不完整的事务也视为已提交。这个值可能会永久损坏数据文件。将`InnoDB`设置为只读。

+   `6` (`SRV_FORCE_NO_LOG_REDO`)

    在恢复过程中不执行重做日志的前进。这个值可能会永久损坏数据文件。将数据库页面留在过时状态，这可能会导致 B 树和其他数据库结构出现更多损坏。将`InnoDB`设置为只读。

您可以从表中`SELECT`以导出它们。当`innodb_force_recovery`值为 3 或更低时，可以`DROP`或`CREATE`表。当`innodb_force_recovery`值大于 3 时，也支持`DROP TABLE`。当`innodb_force_recovery`值大于 4 时，不允许`DROP TABLE`。

如果您知道某个表在回滚时导致意外退出，您可以将其删除。如果遇到由于失败的大规模导入或`ALTER TABLE`导致的无限回滚，您可以终止**mysqld**进程，并将`innodb_force_recovery`设置为`3`，以在没有回滚的情况下启动数据库，然后`DROP`掉导致无限回滚的表。

如果表数据中的损坏阻止您转储整个表内容，带有`ORDER BY *primary_key* DESC`子句的查询可能能够转储受损部分之后的表部分。

如果需要较高的`innodb_force_recovery`值才能启动`InnoDB`，可能存在损坏的数据结构，可能导致复杂查询（包含`WHERE`、`ORDER BY`或其他子句的查询）失败。在这种情况下，您可能只能运行基本的`SELECT * FROM t`查询。
