> 原文：[`dev.mysql.com/doc/refman/8.0/en/ipv6-remote-connections.html`](https://dev.mysql.com/doc/refman/8.0/en/ipv6-remote-connections.html)

#### 7.1.13.4 使用 IPv6 非本地主机地址进行连接

以下过程展示了如何配置 MySQL 以允许远程客户端进行 IPv6 连接。与本地客户端的前面过程类似，但服务器和客户端主机是不同的，并且每个主机都有自己的非本地 IPv6 地址。示例使用以下地址：

```sql
Server host: 2001:db8:0:f101::1
Client host: 2001:db8:0:f101::2
```

这些地址是从[IANA](http://www.iana.org/assignments/ipv6-unicast-address-assignments/ipv6-unicast-address-assignments.xml)推荐的非路由地址范围中选择的，用于文档目的并足以在本地网络上进行测试。要接受来自本地网络之外客户端的 IPv6 连接，服务器主机必须具有公共地址。如果您的网络提供商分配给您一个 IPv6 地址，您可以使用该地址。否则，获取地址的另一种方法是使用 IPv6 代理；请参阅 Section 7.1.13.5, “Obtaining an IPv6 Address from a Broker”。

1.  使用适当的`bind_address`设置启动 MySQL 服务器，以允许其接受 IPv6 连接。例如，在服务器选项文件中放入以下行并重新启动服务器：

    ```sql
    [mysqld]
    bind_address = *
    ```

    将 *（或 `::`）指定为`bind_address`的值，允许所有服务器主机 IPv4 和 IPv6 接口上的 IPv4 和 IPv6 连接。如果要将服务器绑定到特定地址列表，您可以在 MySQL 8.0.13 中通过为`bind_address`指定逗号分隔的值来实现。此示例指定了一个 IPv4 地址以及所需的服务器主机 IPv6 地址：

    ```sql
    [mysqld]
    bind_address = 198.51.100.20,2001:db8:0:f101::1
    ```

    欲了解更多信息，请参阅 Section 7.1.8, “Server System Variables”中的`bind_address`描述。

1.  在服务器主机（`2001:db8:0:f101::1`）上，为可以从客户端主机（`2001:db8:0:f101::2`）连接的用户创建一个帐户：

    ```sql
    mysql> CREATE USER 'remoteipv6user'@'2001:db8:0:f101::2' IDENTIFIED BY 'remoteipv6pass';
    ```

1.  在客户端主机（`2001:db8:0:f101::2`）上，调用**mysql**客户端以使用新帐户连接到服务器：

    ```sql
    $> mysql -h 2001:db8:0:f101::1 -u remoteipv6user -premoteipv6pass
    ```

1.  尝试一些简单的语句，显示连接信息：

    ```sql
    mysql> STATUS
    ...
    Connection:   2001:db8:0:f101::1 via TCP/IP
    ...

    mysql> SELECT CURRENT_USER(), @@bind_address;
    +-----------------------------------+----------------+
    | CURRENT_USER()                    | @@bind_address |
    +-----------------------------------+----------------+
    | remoteipv6user@2001:db8:0:f101::2 | ::             |
    +-----------------------------------+----------------+
    ```
