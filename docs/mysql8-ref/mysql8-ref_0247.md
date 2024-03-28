> 原文：[`dev.mysql.com/doc/refman/8.0/en/ipv6-server-config.html`](https://dev.mysql.com/doc/refman/8.0/en/ipv6-server-config.html)

#### 7.1.13.2 配置 MySQL 服务器以允许 IPv6 连接

MySQL 服务器在一个或多个网络套接字上监听 TCP/IP 连接。每个套接字绑定到一个地址，但一个地址可能映射到多个网络接口。

在服务器启动时设置`bind_address`系统变量，以指定服务器实例接受的 TCP/IP 连接。从 MySQL 8.0.13 开始，您可以为此选项指定多个值，包括任何组合的 IPv6 地址、IPv4 地址和解析为 IPv6 或 IPv4 地址的主机名。或者，您可以指定允许监听多个网络接口的通配符地址格式之一。值为 *（默认值）或值为 `::` 允许在所有服务器主机的 IPv4 和 IPv6 接口上进行 IPv4 和 IPv6 连接。有关更多信息，请参阅第 7.1.8 节，“服务器系统变量”中的`bind_address`描述。
