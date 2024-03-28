> 原文：[`dev.mysql.com/doc/refman/8.0/en/replica-logs-relaylog.html`](https://dev.mysql.com/doc/refman/8.0/en/replica-logs-relaylog.html)

#### 19.2.4.1 中继日志

中继日志与二进制日志类似，由一组包含描述数据库更改事件的编号文件和一个包含所有已使用中继日志文件名称的索引文件组成。中继日志文件的默认位置是数据目录。

术语“中继日志文件”通常表示包含数据库事件的单个编号文件。术语“中继日志”集体表示一组编号的中继日志文件和索引文件。

中继日志文件与二进制日志文件具有相同的格式，并且可以使用**mysqlbinlog**进行读取（参见第 6.6.9 节，“mysqlbinlog — 用于处理二进制日志文件的实用程序”）。如果使用二进制日志事务压缩（从 MySQL 8.0.20 开始提供），写入中继日志的事务有效载荷将以与二进制日志相同的方式进行压缩。有关二进制日志事务压缩的更多信息，请参见第 7.4.4.5 节，“二进制日志事务压缩”。

对于默认复制通道，中继日志文件名采用默认形式`*`host_name`*-relay-bin.*`nnnnnn`*`，其中*`host_name`*是复制服务器主机的名称，*`nnnnnn`*是一个序列号。连续的中继日志文件使用连续的序列号创建，从`000001`开始。对于非默认复制通道，基本名称为`*`host_name`*-relay-bin-*`channel`*`，其中*`channel`*是中继日志中记录的复制通道的名称。

复制使用索引文件来跟踪当前正在使用的中继日志文件。默认的中继日志索引文件名为`*`host_name`*-relay-bin.index`，用于默认通道，非默认复制通道使用`*`host_name`*-relay-bin-*`channel`*.index`。

默认的中继日志文件和中继日志索引文件的名称和位置可以分别通过`relay_log`和`relay_log_index`系统变量进行覆盖（参见第 19.1.6 节，“复制和二进制日志选项和变量”）。

如果一个复制使用默认基于主机的中继日志文件名，那么在设置复制后更改复制的主机名可能会导致复制失败，并显示错误消息“Failed to open the relay log”和“Could not find target log during relay log initialization”。这是一个已知问题（参见 Bug #2122）。如果预计将来可能更改复制的主机名（例如，如果在复制上设置了网络，以便可以使用 DHCP 修改其主机名），则可以通过在初始设置复制时显式使用`relay_log`和`relay_log_index`系统变量来指定中继日志文件名，从而完全避免此问题。这样可以使名称与服务器主机名更改无关。

如果在复制已经开始后遇到问题，解决方法之一是停止复制服务器，将旧中继日志索引文件的内容添加到新文件中，然后重新启动复制。在 Unix 系统上，可以按照以下方式执行：

```sql
$> cat *new_relay_log_name*.index >> *old_relay_log_name*.index
$> mv *old_relay_log_name*.index *new_relay_log_name*.index
```

复制服务器在以下情况下创建一个新的中继日志文件：

+   每次复制 I/O（接收器）线程启动时。

+   当日志被刷新时（例如，使用`FLUSH LOGS`或**mysqladmin flush-logs**）。

+   当当前中继日志文件的大小变得太大时，判定如下：

    +   如果`max_relay_log_size`的值大于 0，则为最大中继日志文件大小。

    +   如果`max_relay_log_size`的值为 0，则`max_binlog_size`确定最大中继日志文件大小。

复制 SQL（应用程序）线程在执行完文件中的所有事件并不再需要该文件时会自动删除每个中继日志文件。没有显式的机制用于删除中继日志，因为复制 SQL 线程会负责执行此操作。然而，`FLUSH LOGS`会旋转中继日志，影响复制 SQL 线程何时删除它们。
