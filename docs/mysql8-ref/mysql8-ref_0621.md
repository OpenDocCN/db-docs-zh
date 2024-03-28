# 10.11.3 并发插入

> 原文：[`dev.mysql.com/doc/refman/8.0/en/concurrent-inserts.html`](https://dev.mysql.com/doc/refman/8.0/en/concurrent-inserts.html)

`MyISAM` 存储引擎支持并发插入，以减少读者和写者在给定表中的竞争：如果 `MyISAM` 表在数据文件中没有空洞（中间删除的行），则可以执行 `INSERT` 语句，同时向表的末尾添加行，而 `SELECT` 语句正在从表中读取行。如果有多个 `INSERT` 语句，则它们会被排队并按顺序执行，与 `SELECT` 语句并发执行。并发 `INSERT` 的结果可能不会立即可见。

`concurrent_insert` 系统变量可用于修改并发插入处理。默认情况下，该变量设置为 `AUTO`（或 1），并发插入会按照上述描述进行处理。如果 `concurrent_insert` 设置为 `NEVER`（或 0），则禁用并发插入。如果该变量设置为 `ALWAYS`（或 2），即使对于具有已删除行的表，也允许在表的末尾进行并发插入。另请参阅 `concurrent_insert` 系统变量的描述。

如果您正在使用二进制日志，对于 `CREATE ... SELECT` 或 `INSERT ... SELECT` 语句，将并发插入转换为普通插入。这样做是为了确保您可以通过在备份操作期间应用日志来重新创建表的精确副本。请参阅 Section 7.4.4, “The Binary Log”。此外，对于这些语句，会在所选表上放置读锁，以阻止对该表的插入。其效果是，该表的并发插入也必须等待。

使用 `LOAD DATA`，如果您在满足并发插入条件的 `MyISAM` 表中指定 `CONCURRENT`（即，它在中间不包含空闲块），则其他会话可以在 `LOAD DATA` 执行时从表中检索数据。即使没有其他会话同时使用该表，使用 `CONCURRENT` 选项也会稍微影响 `LOAD DATA` 的性能。

如果您指定 `HIGH_PRIORITY`，它会覆盖 `--low-priority-updates` 选项的效果，如果服务器是以该选项启动的。它还会导致不使用并发插入。

对于`LOCK TABLE`，`READ LOCAL`和`READ`之间的区别在于，`READ LOCAL`允许非冲突的`INSERT`语句（并发插入）在持有锁的情况下执行。然而，如果在持有锁的同时要使用外部进程来操作数据库，则不能使用此选项。
