> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-cpudata-1sec.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-cpudata-1sec.html)

#### 25.6.16.14 ndbinfo cpudata_1sec 表

`cpudata_1sec`表提供了过去 20 秒内每秒的 CPU 使用情况数据。

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

    在用户空间中花费的 CPU 时间

+   `cpu_idle_time`

    空闲时花费的 CPU 时间

+   `cpu_system_time`

    在系统时间中花费的 CPU 时间

+   `cpu_interrupt_time`

    处理中断（硬件和软件）所花费的 CPU 时间

+   `cpu_exec_vm_time`

    在虚拟机执行中花费的 CPU 时间

+   `elapsed_time`

    用于此测量的微秒时间

##### 注意事项

`cpudata_1sec`表仅在 Linux 和 Solaris 操作系统上可用。

该表在 NDB 8.0.23 中添加。
