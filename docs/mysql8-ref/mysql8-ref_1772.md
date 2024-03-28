> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-cpustat-20sec.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-cpustat-20sec.html)

#### 25.6.16.21 `ndbinfo cpustat_20sec` 表

`cpustat_20sec` 表提供每 20 秒获取一次的原始、每个运行在 `NDB` 内核中的线程的 CPU 数据。

像`cpustat_50ms`和`cpustat_1sec`一样，此表显示每个线程的 20 个测量集，每个集合引用命名持续时间的一个时期。因此，`cpsustat_20sec` 提供了 400 秒的历史记录。

`cpustat_20sec` 表包含以下列：

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

    线程睡眠时间

+   `spin_time`

    线程旋转时间

+   `send_time`

    线程发送时间

+   `buffer_full_time`

    线程缓冲区满时间

+   `elapsed_time`

    经过时间
