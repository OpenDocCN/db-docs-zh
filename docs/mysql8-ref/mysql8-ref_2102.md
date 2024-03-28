> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-socket-summary-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-socket-summary-tables.html)

#### 29.12.20.9 套接字汇总表

这些套接字汇总表聚合计时器和字节计数信息，用于套接字操作：

+   `socket_summary_by_event_name`：聚合由`wait/io/socket/*`工具生成的计时器和字节计数统计信息，适用于所有套接字 I/O 操作，每个套接字工具。

+   `socket_summary_by_instance`：聚合由`wait/io/socket/*`工具生成的计时器和字节计数统计信息，适用于所有套接字 I/O 操作，每个套接字实例。当连接终止时，对应于其的`socket_summary_by_instance`中的行将被删除。

套接字汇总表不会聚合由`idle`事件生成的等待，而套接字在等待来自客户端的下一个请求时。对于`idle`事件的聚合，使用等待事件汇总表；参见第 29.12.20.1 节，“等待事件汇总表”。

每个套接字汇总表都有一个或多个分组列，指示表如何聚合事件。事件名称指的是`setup_instruments`表中事件工具的名称：

+   `socket_summary_by_event_name`有一个`EVENT_NAME`列。每行汇总给定事件名称的事件。

+   `socket_summary_by_instance`有一个`OBJECT_INSTANCE_BEGIN`列。每行汇总给定对象的事件。

每个套接字汇总表都有这些包含聚合值的汇总列：

+   `COUNT_STAR`, `SUM_TIMER_WAIT`, `MIN_TIMER_WAIT`, `AVG_TIMER_WAIT`, `MAX_TIMER_WAIT`

    这些列聚合所有操作。

+   `COUNT_READ`, `SUM_TIMER_READ`, `MIN_TIMER_READ`, `AVG_TIMER_READ`, `MAX_TIMER_READ`, `SUM_NUMBER_OF_BYTES_READ`

    这些列聚合所有接收操作（`RECV`，`RECVFROM`和`RECVMSG`）。

+   `COUNT_WRITE`, `SUM_TIMER_WRITE`, `MIN_TIMER_WRITE`, `AVG_TIMER_WRITE`, `MAX_TIMER_WRITE`, `SUM_NUMBER_OF_BYTES_WRITE`

    这些列聚合所有发送操作（`SEND`，`SENDTO`和`SENDMSG`）。

+   `COUNT_MISC`, `SUM_TIMER_MISC`, `MIN_TIMER_MISC`, `AVG_TIMER_MISC`, `MAX_TIMER_MISC`

    这些列聚合所有其他套接字操作，如`CONNECT`，`LISTEN`，`ACCEPT`，`CLOSE`和`SHUTDOWN`。这些操作没有字节计数。

`socket_summary_by_instance`表还具有一个`EVENT_NAME`列，指示套接字的类别：`client_connection`、`server_tcpip_socket`、`server_unix_socket`。可以根据此列进行分组，例如将客户端活动与服务器监听套接字的活动隔离开来。

套接字摘要表具有以下索引：

+   `socket_summary_by_event_name`:

    +   主键为(`EVENT_NAME`)

+   `socket_summary_by_instance`:

    +   主键为(`OBJECT_INSTANCE_BEGIN`)

    +   索引为(`EVENT_NAME`)

允许对套接字摘要表执行`TRUNCATE TABLE`。除了`events_statements_summary_by_digest`之外，它将摘要列重置为零，而不是删除行。
