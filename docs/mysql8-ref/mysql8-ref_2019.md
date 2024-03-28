# 29.12.3 性能模式实例表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-instance-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-instance-tables.html)

29.12.3.1 条件实例表

29.12.3.2 文件实例表

29.12.3.3 互斥实例表

29.12.3.4 读写锁实例表

29.12.3.5 套接字实例表

实例表记录了被检测的对象类型。它们提供事件名称和解释说明或状态信息：

+   `cond_instances`：条件同步对象实例

+   `file_instances`：文件实例

+   `mutex_instances`：互斥同步对象实例

+   `rwlock_instances`：锁同步对象实例

+   `socket_instances`：活动连接实例

这些表列出了被检测的同步对象、文件和连接。同步对象有三种类型：`cond`、`mutex` 和 `rwlock`。每个实例表都有一个 `EVENT_NAME` 或 `NAME` 列，用于指示与每行关联的仪器。仪器名称可能有多个部分，并形成一个层次结构，如 第 29.6 节，“性能模式仪器命名约定” 中所讨论的。

`mutex_instances.LOCKED_BY_THREAD_ID` 和 `rwlock_instances.WRITE_LOCKED_BY_THREAD_ID` 列对于调查性能瓶颈或死锁非常重要。有关如何使用它们进行此目的的示例，请参见 第 29.19 节，“使用性能模式诊断问题”
