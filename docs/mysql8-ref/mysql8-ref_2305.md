> 原文：[`dev.mysql.com/doc/refman/8.0/en/packet-too-large.html`](https://dev.mysql.com/doc/refman/8.0/en/packet-too-large.html)

#### B.3.2.8 Packet Too Large

通信数据包是发送到 MySQL 服务器的单个 SQL 语句，发送到客户端的单个行，或从复制源服务器发送到副本的二进制日志事件。

可以传输到 MySQL 8.0 服务器或客户端的最大可能数据包为 1GB。

当 MySQL 客户端或**mysqld**服务器接收到大于`max_allowed_packet`字节的数据包时，会发出`ER_NET_PACKET_TOO_LARGE`错误并关闭连接。如果通信数据包过大，有些客户端还可能会出现`Lost connection to MySQL server during query`错误。

客户端和服务器都有自己的`max_allowed_packet`变量，因此如果要处理大数据包，必须在客户端和服务器中都增加此变量。

如果你正在使用**mysql**客户端程序，其默认`max_allowed_packet`变量为 16MB。要设置更大的值，请像这样启动**mysql**：

```sql
$> mysql --max_allowed_packet=32M
```

这将数据包大小设置为 32MB。

服务器的默认`max_allowed_packet`值为 64MB。如果服务器需要处理大查询（例如，如果您正在使用大`BLOB`列），可以增加此值。例如，要将变量设置为 128MB，请像这样启动服务器：

```sql
$> mysqld --max_allowed_packet=128M
```

您还可以使用选项文件来设置`max_allowed_packet`。例如，要为服务器设置为 128MB 的大小，请在选项文件中添加以下行：

```sql
[mysqld]
max_allowed_packet=128M
```

增加此变量的值是安全的，因为只有在需要时才分配额外的内存。例如，**mysqld**仅在发出长查询或**mysqld**必须返回大结果行时才分配更多内存。变量的小默认值是为了捕获客户端和服务器之间的不正确数据包，并确保您不会因意外使用大数据包而耗尽内存。

如果您正在使用大型`BLOB`值，但没有为**mysqld**分配足够的内存来处理查询，也可能会遇到大数据包的奇怪问题。 如果您怀疑是这种情况，请尝试在**mysqld_safe**脚本的开头添加**ulimit -d 256000**，然后重新启动**mysqld**。
