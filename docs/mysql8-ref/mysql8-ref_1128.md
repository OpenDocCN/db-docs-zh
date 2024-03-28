> 原文：[`dev.mysql.com/doc/refman/8.0/en/shutdown.html`](https://dev.mysql.com/doc/refman/8.0/en/shutdown.html)

#### 15.7.8.9 关闭语句

```sql
SHUTDOWN
```

此语句停止 MySQL 服务器。它需要`SHUTDOWN` 权限。

`SHUTDOWN` 提供了一个 SQL 级别的接口，可以使用 **mysqladmin shutdown** 命令或 `mysql_shutdown()` C API 函数来实现相同的功能。成功的 `SHUTDOWN` 序列包括检查权限、验证参数，并向客户端发送一个 OK 数据包。然后服务器关闭。

`Com_shutdown` 状态变量跟踪 `SHUTDOWN` 语句的数量。因为状态变量在每次服务器启动时初始化，并且在重新启动时不会保留，所以 `Com_shutdown` 通常值为零，但如果执行了但失败了 `SHUTDOWN` 语句，则可能为非零。

另一种停止服务器的方法是发送一个 `SIGTERM` 信号，可以由 `root` 或拥有服务器进程的帐户执行。`SIGTERM` 使得可以在不连接到服务器的情况下执行服务器关闭。参见 第 6.10 节，“MySQL 中的 Unix 信号处理”。
