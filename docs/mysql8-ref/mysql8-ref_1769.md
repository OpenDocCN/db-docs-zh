> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-cpustat.html`](https://dev.mysql.com/doc/refman/8.0/en/mysql-cluster-ndbinfo-cpustat.html)

#### 25.6.16.18 ndbinfo cpustat 表

`cpustat`表每秒提供每个线程在`NDB`内核中运行的每个线程的 CPU 统计信息。

`cpustat`表包含以下列：

+   `node_id`

    线程所在节点的 ID

+   `thr_no`

    线程 ID（特定于此节点）

+   `OS_user`

    操作系统用户时间

+   `OS_system`

    操作系统系统时间

+   `OS_idle`

    操作系统空闲时间

+   `thread_exec`

    线程执行时间

+   `thread_sleeping`

    线程睡眠时间

+   `thread_spinning`

    线程旋转时间

+   `thread_send`

    线程发送时间

+   `thread_buffer_full`

    线程缓冲区满时间

+   `elapsed_time`

    经过的时间
