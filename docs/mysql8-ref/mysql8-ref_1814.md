> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-threadstat.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-threadstat.html)

#### 25.6.16.63 `ndbinfo threadstat` 表

`threadstat` 表提供了运行在 `NDB` 内核中的线程统计的大致快照。

`threadstat` 表包含以下列：

+   `node_id`

    节点 ID

+   `thr_no`

    线程 ID

+   `thr_nm`

    线程名称

+   `c_loop`

    主循环中的循环次数

+   `c_exec`

    执行的信号数

+   `c_wait`

    等待额外输入的次数

+   `c_l_sent_prioa`

    发送到自身节点的优先级 A 信号数

+   `c_l_sent_priob`

    发送到自身节点的优先级 B 信号数

+   `c_r_sent_prioa`

    发送到远程节点的优先级 A 信号数

+   `c_r_sent_priob`

    发送到远程节点的优先级 B 信号数

+   `os_tid`

    OS 线程 ID

+   `os_now`

    OS 时间（毫秒）

+   `os_ru_utime`

    OS 用户 CPU 时间（微秒）

+   `os_ru_stime`

    OS 系统 CPU 时间（微秒）

+   `os_ru_minflt`

    OS 页面回收（软页错误）

+   `os_ru_majflt`

    OS 页面错误（硬页错误）

+   `os_ru_nvcsw`

    OS 自愿上下文切换

+   `os_ru_nivcsw`

    OS 非自愿上下文切换

##### 注意事项

`os_time` 使用系统 `gettimeofday()` 调用。

`os_ru_utime`、`os_ru_stime`、`os_ru_minflt`、`os_ru_majflt`、`os_ru_nvcsw` 和 `os_ru_nivcsw` 列的值是使用系统 `getrusage()` 调用或等效调用获取的。

由于该表包含在特定时间点采取的计数，为了获得最佳结果，需要定期查询该表并将结果存储在一个中间表或多个表中。 MySQL 服务器的事件调度程序可用于自动化此类监控。有关更多信息，请参见 Section 27.4, “Using the Event Scheduler”。
