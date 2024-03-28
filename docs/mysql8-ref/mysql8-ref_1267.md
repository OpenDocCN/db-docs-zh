# 17.17.1 InnoDB 监视器类型

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-monitor-types.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-monitor-types.html)

有两种类型的`InnoDB`监视器：

+   标准`InnoDB`监视器显示以下类型的信息：

    +   主后台线程完成的工作

    +   信号量等待

    +   最近的外键和死锁错误数据

    +   事务的锁等待

    +   活动事务持有的表和记录锁

    +   待处理的 I/O 操作及相关统计信息

    +   插入缓冲区和自适应哈希索引统计信息

    +   重做日志数据

    +   缓冲池统计信息

    +   行操作数据

+   `InnoDB`锁监视器会在标准`InnoDB`监视器输出中打印额外的锁信息。
