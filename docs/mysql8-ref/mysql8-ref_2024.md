> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-socket-instances-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-socket-instances-table.html)

#### 29.12.3.5 The socket_instances Table

`socket_instances`表提供了 MySQL 服务器的活动连接的实时快照。该表每个 TCP/IP 或 Unix 套接字文件连接包含一行。此表中可用的信息提供了服务器的活动连接的实时快照。（有关套接字摘要表的其他信息，请参见第 29.12.20.9 节“套接字摘要表”）。

```sql
mysql> SELECT * FROM performance_schema.socket_instances\G
*************************** 1\. row ***************************
           EVENT_NAME: wait/io/socket/sql/server_unix_socket
OBJECT_INSTANCE_BEGIN: 4316619408
            THREAD_ID: 1
            SOCKET_ID: 16
                   IP:
                 PORT: 0
                STATE: ACTIVE
*************************** 2\. row ***************************
           EVENT_NAME: wait/io/socket/sql/client_connection
OBJECT_INSTANCE_BEGIN: 4316644608
            THREAD_ID: 21
            SOCKET_ID: 39
                   IP: 127.0.0.1
                 PORT: 55233
                STATE: ACTIVE
*************************** 3\. row ***************************
           EVENT_NAME: wait/io/socket/sql/server_tcpip_socket
OBJECT_INSTANCE_BEGIN: 4316699040
            THREAD_ID: 1
            SOCKET_ID: 14
                   IP: 0.0.0.0
                 PORT: 50603
                STATE: ACTIVE
```

Socket 工具的名称形式为`wait/io/socket/sql/*`socket_type`*`，使用方式如下：

1.  服务器为其支持的每种网络协议都有一个监听套接字。与 TCP/IP 或 Unix 套接字文件连接的监听套接字相关联的工具具有`server_tcpip_socket`或`server_unix_socket`的*`socket_type`*值。

1.  当监听套接字检测到连接时，服务器将连接转移到由单独线程管理的新套接字。新连接线程的工具具有`client_connection`的*`socket_type`*值。

1.  当连接终止时，对应的`socket_instances`表中的行将被删除。

`socket_instances`表具有以下列：

+   `EVENT_NAME`

    产生事件的`wait/io/socket/*`工具的名称。这是`setup_instruments`表中的一个`NAME`值。工具名称可能由多个部分组成，形成一个层次结构，如第 29.6 节“性能模式工具命名约定”中所讨论的那样。

+   `OBJECT_INSTANCE_BEGIN`

    此列唯一标识套接字。该值是内存中对象的地址。

+   `THREAD_ID`

    服务器分配的内部线程标识符。每个套接字由单个线程管理，因此每个套接字可以映射到一个线程，该线程可以映射到一个服务器进程。

+   `SOCKET_ID`

    分配给套接字的内部文件句柄。

+   `IP`

    客户端 IP 地址。该值可以是 IPv4 或 IPv6 地址，或者为空以指示 Unix 套接字文件连接。

+   `PORT`

    TCP/IP 端口号，范围从 0 到 65535。

+   `STATE`

    套接字状态，要么是 `IDLE`，要么是 `ACTIVE`。活动套接字的等待时间使用相应的套接字工具进行跟踪。空闲套接字的等待时间使用 `idle` 工具进行跟踪。

    如果套接字正在等待来自客户端的请求，则套接字处于空闲状态。当套接字变为空闲时，跟踪套接字的 `socket_instances` 表中的事件行从 `ACTIVE` 状态切换到 `IDLE`。`EVENT_NAME` 值仍为 `wait/io/socket/*`，但工具的计时被暂停。相反，在 `events_waits_current` 表中生成一个 `EVENT_NAME` 值为 `idle` 的事件。

    当接收到下一个请求时，`idle` 事件终止，套接字实例从 `IDLE` 切换到 `ACTIVE`，套接字工具的计时恢复。

`socket_instances` 表具有以下索引：

+   在 (`OBJECT_INSTANCE_BEGIN`) 上建立主键

+   在 (`THREAD_ID`) 上建立索引

+   在 (`SOCKET_ID`) 上建立索引

+   在 (`IP`, `PORT`) 上建立索引

不允许对 `socket_instances` 表使用 `TRUNCATE TABLE`。

`IP:PORT` 列组合值标识连接。此组合值在 `events_waits_*`xxx`*` 表的 `OBJECT_NAME` 列中使用，用于标识套接字事件的连接来源：

+   对于 Unix 域监听器套接字 (`server_unix_socket`)，端口为 0，IP 为 `''`。

+   对通过 Unix 域监听器 (`client_connection`) 的客户端连接，端口为 0，IP 为 `''`。

+   对于 TCP/IP 服务器监听器套接字 (`server_tcpip_socket`)，端口始终是主端口（例如，3306），IP 始终是 `0.0.0.0`。

+   对通过 TCP/IP 监听器 (`client_connection`) 的客户端连接，端口是服务器分配的任何端口，但绝不是 0。IP 是发起主机的 IP (`127.0.0.1` 或 `::1` 为本地主机)
