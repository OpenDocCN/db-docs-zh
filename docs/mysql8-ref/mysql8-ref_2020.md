> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-cond-instances-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-cond-instances-table.html)

#### 29.12.3.1 cond_instances 表

`cond_instances` 表列出了服务器执行时性能模式看到的所有条件。条件是代码中使用的同步机制，用于发出特定事件已发生的信号，以便等待此条件的线程可以恢复工作。

当一个线程在等待某些事件发生时，条件名称表明了线程在等待什么，但没有直接的方法可以告诉是哪个其他线程或哪些线程导致了条件的发生。

`cond_instances` 表具有以下列：

+   `NAME`

    与条件相关联的工具名称。

+   `OBJECT_INSTANCE_BEGIN`

    工具化条件在内存中的地址。

`cond_instances` 表具有以下索引：

+   在 (`OBJECT_INSTANCE_BEGIN`) 上的主键

+   在 (`NAME`) 上的索引

`TRUNCATE TABLE` 不允许用于 `cond_instances` 表。
