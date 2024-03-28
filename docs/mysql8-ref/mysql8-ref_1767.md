> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-cpudata-50ms.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-cpudata-50ms.html)

#### 25.6.16.16 ndbinfo cpudata_50ms 表

`cpudata_50ms`表提供了关于过去一秒钟内每 50 毫秒间隔的 CPU 使用情况的数据。

`cpustat`表包含以下列：

+   `node_id`

    节点 ID

+   `measurement_id`

    测量序列 ID；后续测量具有较低的 ID

+   `cpu_no`

    CPU ID

+   `cpu_online`

    如果 CPU 当前在线，则为 1，否则为 0

+   `cpu_userspace_time`

    CPU 时间花费在用户空间上

+   `cpu_idle_time`

    CPU 空闲时间

+   `cpu_system_time`

    CPU 时间花费在系统时间上

+   `cpu_interrupt_time`

    CPU 时间花费在处理中断（硬件和软件）上

+   `cpu_exec_vm_time`

    CPU 时间花费在虚拟机执行上

+   `elapsed_time`

    用于此测量的微秒时间

##### 注释

`cpudata_50ms`表仅在 Linux 和 Solaris 操作系统上可用。

这个表是在 NDB 8.0.23 中添加的。
