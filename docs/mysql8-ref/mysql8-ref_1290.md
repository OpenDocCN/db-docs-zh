# 17.21 InnoDB 故障排除

> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-troubleshooting.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-troubleshooting.html)

17.21.1 故障排除 InnoDB I/O 问题

17.21.2 故障排除恢复失败

17.21.3 强制 InnoDB 恢复

17.21.4 故障排除 InnoDB 数据字典操作

17.21.5 InnoDB 错误处理

以下一般准则适用于故障排除 `InnoDB` 问题：

+   当操作失败或怀疑存在 bug 时，请查看 MySQL 服务器错误日志（参见 7.4.2 “错误日志”）。服务器错误消息参考 提供了一些常见的 `InnoDB` 特定错误的故障排除信息。

+   如果故障与死锁有关，请启用 `innodb_print_all_deadlocks` 选项运行，以便将每个死锁的详细信息打印到 MySQL 服务器错误日志中。有关死锁的信息，请参见 17.7.5 “InnoDB 中的死锁”。

+   如果问题与 `InnoDB` 数据字典有关，请参见 17.21.4 “故障排除 InnoDB 数据字典操作”。

+   在故障排除时，通常最好从命令提示符下运行 MySQL 服务器，而不是通过 **mysqld_safe** 或作为 Windows 服务运行。然后您可以看到 **mysqld** 打印到控制台的内容，从而更好地了解发生了什么。在 Windows 上，使用 `--console` 选项启动 **mysqld**，将输出定向到控制台窗口。

+   启用 `InnoDB` Monitors 以获取有关问题的信息（参见 17.17 “InnoDB Monitors”）。如果问题与性能有关，或者服务器似乎挂起，您应该启用标准 Monitor 以打印有关 `InnoDB` 内部状态的信息。如果问题与锁有关，请启用 Lock Monitor。如果问题与表创建、表空间或数据字典操作有关，请参考 InnoDB 信息模式系统表 来检查 `InnoDB` 内部数据字典的内容。

    `InnoDB` 在以下情况下临时启用标准 `InnoDB` Monitor 输出：

    +   一个长时间的信号量等待

    +   `InnoDB`在缓冲池中找不到空闲块。

    +   缓冲池超过`67%`被锁堆或自适应哈希索引占用。

+   如果你怀疑某个表损坏，运行`CHECK TABLE`命令检查该表。
