> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-cpudata.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-cpudata.html)

#### 25.6.16.13 ndbinfo cpudata 表

`cpudata` 表提供了最近一秒钟的 CPU 使用情况数据。

`cpustat` 表包含以下列：

+   `node_id`

    节点 ID

+   `cpu_no`

    CPU ID

+   `cpu_online`

    如果 CPU 当前在线，则为 1，否则为 0

+   `cpu_userspace_time`

    CPU 花费在用户空间的时间

+   `cpu_idle_time`

    CPU 空闲时间

+   `cpu_system_time`

    CPU 花费在系统时间的时间

+   `cpu_interrupt_time`

    CPU 花费在处理中断（硬件和软件）的时间

+   `cpu_exec_vm_time`

    CPU 花费在虚拟机执行的时间

##### 注意事项

`cpudata` 表仅在 Linux 和 Solaris 操作系统上可用。

该表在 NDB 8.0.23 版本中添加。
