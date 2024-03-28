> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-cpustat-1sec.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-cpustat-1sec.html)

#### 25.6.16.20 `cpustat_1sec` 表

`cpustat-1sec` 表提供每秒为 `NDB` 内核中运行的每个线程获取的原始的每线程 CPU 数据。

与`cpustat_50ms`和`cpustat_20sec`类似，该表每个线程显示 20 个测量集，每个集合引用一个指定持续时间的周期。因此，`cpsustat_1sec` 提供了 20 秒的历史记录。

`cpustat_1sec` 表包含以下列：

+   `node_id`

    线程所在节点的 ID

+   `thr_no`

    线程 ID（特定于此节点）

+   `OS_user_time`

    操作系统用户时间

+   `OS_system_time`

    操作系统系统时间

+   `OS_idle_time`

    操作系统空闲时间

+   `exec_time`

    线程执行时间

+   `sleep_time`

    线程休眠时间

+   `spin_time`

    线程自旋时间

+   `send_time`

    线程发送时间

+   `buffer_full_time`

    线程缓冲区满时间

+   `elapsed_time`

    经过时间
