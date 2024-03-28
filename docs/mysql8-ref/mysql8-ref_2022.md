> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-mutex-instances-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-mutex-instances-table.html)

#### 29.12.3.3 mutex_instances 表

`mutex_instances` 表列出了服务器执行时性能模式看到的所有互斥锁。互斥锁是代码中用于强制只有一个线程可以访问某些共享资源的同步机制。该资源被互斥锁“保护”。

当两个在服务器中执行的线程（例如，两个用户会话同时执行查询）需要访问相同的资源（文件、缓冲区或某些数据），这两个线程相互竞争，因此第一个查询获得互斥锁的线程会导致另一个查询等待，直到第一个完成并解锁互斥锁。

在持有互斥锁时执行的工作被称为“临界区”，多个查询以串行方式（一次一个）执行此临界区，这可能是一个潜在的瓶颈。

`mutex_instances` 表具有以下列：

+   `NAME`

    与互斥锁关联的仪器名称。

+   `OBJECT_INSTANCE_BEGIN`

    仪器化互斥锁在内存中的地址。

+   `LOCKED_BY_THREAD_ID`

    当一个线程当前持有一个互斥锁时，`LOCKED_BY_THREAD_ID` 是锁定线程的 `THREAD_ID`，否则为 `NULL`。

`mutex_instances` 表具有以下索引：

+   主键为 (`OBJECT_INSTANCE_BEGIN`)

+   索引为 (`NAME`)

+   索引为 (`LOCKED_BY_THREAD_ID`)

`TRUNCATE TABLE` 不允许用于 `mutex_instances` 表。

对于代码中仪器化的每个互斥锁，性能模式提供以下信息。

+   `setup_instruments` 表列出了仪器点的名称，带有前缀 `wait/synch/mutex/`。

+   当一些代码创建一个互斥锁时，会向 `mutex_instances` 表添加一行。`OBJECT_INSTANCE_BEGIN` 列是一个唯一标识互斥锁的属性。

+   当一个线程尝试锁定一个互斥锁时，`events_waits_current` 表显示了一个针对该线程的行，指示它正在等待一个互斥锁（在 `EVENT_NAME` 列中），并指示正在等待的互斥锁是哪个（在 `OBJECT_INSTANCE_BEGIN` 列中）。

+   当一个线程成功锁定一个互斥锁时：

    +   `events_waits_current`显示互斥锁上的等待已完成（在`TIMER_END`和`TIMER_WAIT`列中）

    +   完成的等待事件被添加到`events_waits_history`和`events_waits_history_long`表中

    +   `mutex_instances`显示该互斥锁现在由该线程拥有（在`THREAD_ID`列中）。

+   当线程解锁互斥锁时，`mutex_instances`显示该互斥锁现在没有所有者（`THREAD_ID`列为`NULL`）。

+   当互斥锁对象被销毁时，相应的行从`mutex_instances`中移除。

通过对上述两个表执行查询，监控应用程序或数据库管理员可以检测涉及互斥锁的线程之间的瓶颈或死锁：

+   `events_waits_current`，查看线程正在等待哪个互斥锁

+   `mutex_instances`，查看哪个其他线程当前拥有互斥锁
