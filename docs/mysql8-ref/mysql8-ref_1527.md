# 20.6.3 保护分布式恢复连接

> 原文：[`dev.mysql.com/doc/refman/8.0/en/group-replication-distributed-recovery-securing.html`](https://dev.mysql.com/doc/refman/8.0/en/group-replication-distributed-recovery-securing.html)

20.6.3.1 为分布式恢复保护用户凭据

20.6.3.2 为分布式恢复配置安全套接字层 (SSL) 连接

重要提示

当使用 MySQL 通信栈（`group_replication_communication_stack=MYSQL`）并且成员之间的连接是安全的（`group_replication_ssl_mode` 未设置为 `DISABLED`）时，本节讨论的安全设置不仅适用于分布式恢复连接，还适用于一般组成员之间的组通信。

当成员加入组时，分布式恢复是通过远程克隆操作（如果可用且适用）和异步复制连接的组合来执行的。有关分布式恢复的详细描述，请参见 Section 20.5.4, “Distributed Recovery”。

在 MySQL 8.0.20 版本之前，组成员根据 MySQL 服务器的 `hostname` 和 `port` 系统变量，为加入成员提供标准的 SQL 客户端连接，用于分布式恢复。从 MySQL 8.0.21 开始，组成员可以为加入成员提供一组备用的分布式恢复端点，作为专用的客户端连接。更多详情请参见 Section 20.5.4.1, “Connections for Distributed Recovery”。请注意，为分布式恢复提供给加入成员的连接 *并非* 用于 Group Replication 在使用 XCom 通信栈进行组通信时在线成员之间通信的连接（`group_replication_communication_stack=XCOM`）。

为了保护组内的分布式恢复连接，请确保复制用户的用户凭据得到妥善保护，并在可能的情况下使用 SSL 进行分布式恢复连接。
