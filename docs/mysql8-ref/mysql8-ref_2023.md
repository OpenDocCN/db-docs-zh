> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-rwlock-instances-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-rwlock-instances-table.html)

#### 29.12.3.4 `rwlock_instances`表

`rwlock_instances`表列出了在服务器执行时性能模式看到的所有 rwlock（读写锁）实例。`rwlock`是代码中使用的同步机制，用于强制在给定时间内的线程可以按照某些规则访问一些共享资源。资源被称为由`rwlock`“保护”。访问可以是共享的（许多线程可以同时拥有读锁），独占的（一次只有一个线程可以拥有写锁），或共享-独占的（一个线程可以拥有写锁，同时允许其他线程进行不一致的读取）。共享-独占访问又称为`sxlock`，可优化并发性并改善读写工作负载的可伸缩性。

根据请求锁的线程数量以及请求的锁的性质，访问可以以共享模式、独占模式、共享-独占模式或根本不授予访问，等待其他线程先完成。

`rwlock_instances`表具有以下列：

+   `NAME`

    与锁相关联的工具名称。

+   `OBJECT_INSTANCE_BEGIN`

    工具锁在内存中的地址。

+   `WRITE_LOCKED_BY_THREAD_ID`

    当一个线程当前以独占（写）模式锁定一个`rwlock`时，`WRITE_LOCKED_BY_THREAD_ID`是锁定线程的`THREAD_ID`，否则为`NULL`。

+   `READ_LOCKED_BY_COUNT`

    当一个线程当前以共享（读）模式锁定一个`rwlock`时，`READ_LOCKED_BY_COUNT`会增加 1。这只是一个计数器，因此不能直接用来找出哪个线程持有读锁，但可以用来查看`rwlock`上是否存在读争用，以及当前有多少读者活跃。

`rwlock_instances`表具有以下索引：

+   主键为(`OBJECT_INSTANCE_BEGIN`)

+   索引为(`NAME`)

+   索引为(`WRITE_LOCKED_BY_THREAD_ID`)

`TRUNCATE TABLE`不允许用于`rwlock_instances`表。

通过对以下两个表执行查询，监控应用程序或 DBA 可以检测到涉及锁的线程之间的一些瓶颈或死锁：

+   `events_waits_current`，查看线程正在等待哪个`rwlock`

+   `rwlock_instances`，查看当前拥有`rwlock`的其他线程。

存在一个限制：`rwlock_instances` 只能用于识别持有写锁的线程，而不能识别持有读锁的线程。
