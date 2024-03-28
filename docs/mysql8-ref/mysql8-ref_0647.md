# 10.14.9 事件调度器线程状态

> 原文：[`dev.mysql.com/doc/refman/8.0/en/event-scheduler-thread-states.html`](https://dev.mysql.com/doc/refman/8.0/en/event-scheduler-thread-states.html)

这些状态发生在事件调度器线程、用于执行计划事件的线程，或终止调度器的线程。

+   `清除`

    调度器线程或正在执行事件的线程正在终止并即将结束。

+   `已初始化`

    调度器线程或执行事件的线程已被初始化。

+   `等待下一次激活`

    调度器有一个非空的事件队列，但下一次激活在未来。

+   `等待调度器停止`

    该线程发出了`SET GLOBAL event_scheduler=OFF`命令，并正在等待调度器停止。

+   `等待空队列`

    调度器的事件队列为空，正在休眠。
