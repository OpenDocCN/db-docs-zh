> 原文：[`dev.mysql.com/doc/refman/8.0/en/mysqlbinlog-server-id.html`](https://dev.mysql.com/doc/refman/8.0/en/mysqlbinlog-server-id.html)

#### 6.6.9.4 指定 mysqlbinlog 服务器 ID

当使用`--read-from-remote-server`选项调用时，**mysqlbinlog**连接到 MySQL 服务器，指定服务器 ID 以标识自身，并从服务器请求二进制日志文件。您可以使用**mysqlbinlog**以几种方式从服务器请求日志文件：

+   指定一个明确命名的文件集：对于每个文件，**mysqlbinlog**连接并发出`Binlog dump`命令。服务器发送文件并断开连接。每个文件都有一个连接。

+   指定起始文件和`--to-last-log`：**mysqlbinlog**连接并发出`Binlog dump`命令以获取所有文件。服务器发送所有文件并断开连接。

+   指定起始文件和`--stop-never`（隐含`--to-last-log`）：**mysqlbinlog**连接并发出`Binlog dump`命令以获取所有文件。服务器发送所有文件，但在发送最后一个文件后不会断开连接。

仅使用`--read-from-remote-server`时，**mysqlbinlog**连接时使用服务器 ID 为 0，告诉服务器在发送最后一个请求的日志文件后断开连接。

使用`--read-from-remote-server`和`--stop-never`，**mysqlbinlog**连接时使用非零的服务器 ID，因此在发送最后一个日志文件后服务器不会断开连接。默认情况下服务器 ID 为 1，但可以使用`--connection-server-id`进行更改。

因此，对于请求文件的前两种方式，服务器会断开连接，因为**mysqlbinlog**指定了服务器 ID 为 0。如果提供了`--stop-never`，则不会断开连接，因为**mysqlbinlog**指定了非零的服务器 ID。
