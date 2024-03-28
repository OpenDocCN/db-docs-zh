> 原文：[`dev.mysql.com/doc/refman/8.0/en/restart.html`](https://dev.mysql.com/doc/refman/8.0/en/restart.html)

#### 15.7.8.8 RESTART Statement

```sql
RESTART
```

此语句停止并重新启动 MySQL 服务器。它需要`SHUTDOWN`权限。

`RESTART`的一个用途是当无法或不方便在服务器主机上获得 MySQL 服务器的命令行访问以重新启动时。例如，可以在运行时使用`SET PERSIST_ONLY`对系统变量进行配置更改，这些变量只能在服务器启动时设置，但服务器仍然必须重新启动才能使这些更改生效。`RESTART`语句提供了一种在客户端会话中执行此操作的方法，而无需在服务器主机上需要命令行访问。

注意

执行`RESTART`语句后，客户端可以预期当前连接将丢失。如果启用了自动重新连接，则在服务器重新启动后重新建立连接。否则，必须手动重新建立连接。

成功执行`RESTART`操作需要**mysqld**在具有可用于检测为重新启动目的而执行的服务器关闭的监控进程的环境中运行：

+   在存在监控进程的情况下，`RESTART`导致**mysqld**终止，以便监控进程可以确定应启动新的**mysqld**实例。

+   如果没有监控进程存在，`RESTART`将失败并显示错误。

这些平台为`RESTART`语句提供了必要的监控支持：

+   Windows，在将**mysqld**作为 Windows 服务或独立运行时。(**mysqld**分叉，一个进程充当监视器，另一个进程充当服务器。)

+   使用 systemd 或**mysqld_safe**管理**mysqld**的 Unix 和类 Unix 系统。

要配置监控环境，使**mysqld**启用`RESTART`语句：

1.  在启动**mysqld**之前，将`MYSQLD_PARENT_PID`环境变量设置为启动**mysqld**的进程的进程 ID 的值。

1.  当**mysqld**由于使用`RESTART`语句而执行关闭时，它会返回退出码 16。

1.  当监控过程检测到退出码为 16 时，它会重新启动**mysqld**。否则，它会退出。

下面是在**bash** shell 中实现的最小示例：

```sql
#!/bin/bash

export MYSQLD_PARENT_PID=$$

export MYSQLD_RESTART_EXIT=16

while true ; do
  bin/mysqld *mysqld options here*
  if [ $? -ne $MYSQLD_RESTART_EXIT ]; then
    break
  fi
done
```

在 Windows 上，用于实现`RESTART`的分叉使得确定要附加到进行调试的服务器进程更加困难。为了缓解这个问题，使用`--gdb`启动服务器会抑制分叉，除了设置调试环境的其他操作。在非调试设置中，可以使用`--no-monitor` 仅用于抑制监控进程的分叉。对于使用`--gdb`或`--no-monitor`启动的服务器，执行`RESTART`会导致服务器简单地退出而不重新启动。

`Com_restart`状态变量跟踪`RESTART`语句的数量。因为状态变量在每次服务器启动时初始化，并且不会跨重启持续存在，`Com_restart`通常值为零，但如果执行了`RESTART`语句但失败了，它可能是非零值。
