# 17.7.2 InnoDB 事务模型

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-transaction-model.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-transaction-model.html)

17.7.2.1 事务隔离级别

17.7.2.2 自动提交、提交和回滚

17.7.2.3 一致性非锁定读取

17.7.2.4 锁定读取

`InnoDB`事务模型旨在将多版本数据库的最佳特性与传统的两阶段锁定相结合。`InnoDB`在行级别执行锁定，并默认以 Oracle 风格的非锁定一致性读取运行查询。`InnoDB`中的锁信息以节省空间的方式存储，因此不需要锁升级。通常，允许多个用户锁定`InnoDB`表中的每一行，或任意随机子集的行，而不会导致`InnoDB`内存耗尽。
