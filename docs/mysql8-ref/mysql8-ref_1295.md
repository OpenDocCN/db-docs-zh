# 17.21.5 InnoDB 错误处理

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-error-handling.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-error-handling.html)

以下项目描述了`InnoDB`如何执行错误处理。`InnoDB`有时只回滚失败的语句，有时会回滚整个事务。

+   如果在表空间中耗尽文件空间，则会发生 MySQL `Table is full`错误，并且`InnoDB`会回滚 SQL 语句。

+   事务死锁会导致`InnoDB`回滚整个事务。发生这种情况时，请重试整个事务。

    锁等待超时会导致`InnoDB`回滚当前语句（等待锁并遇到超时的语句）。要使整个事务回滚，请启动服务器时启用`--innodb-rollback-on-timeout`。如果使用默认行为，请重试语句，或者如果启用了`--innodb-rollback-on-timeout`，则重试整个事务。

    在繁忙服务器上，死锁和锁等待超时是正常的，应用程序需要意识到它们可能发生，并通过重试来处理。通过在事务期间对数据进行第一次更改和提交之间尽可能少地进行工作，可以减少发生这些情况的可能性，从而锁定的时间最短，影响的行数最少。有时将工作分割到不同的事务中可能是实用和有帮助的。

+   如果发生重复键错误，则回滚 SQL 语句，如果在语句中未指定`IGNORE`选项。

+   `行过长错误`会回滚 SQL 语句。

+   其他错误大多由 MySQL 代码层（在`InnoDB`存储引擎层之上）检测到，并回滚相应的 SQL 语句。在回滚单个 SQL 语句时不会释放锁。

在隐式回滚期间，以及在执行显式`ROLLBACK` SQL 语句期间，`SHOW PROCESSLIST`在相关连接的`State`列中显示`Rolling back`。
