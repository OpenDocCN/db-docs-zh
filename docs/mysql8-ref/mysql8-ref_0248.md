> 原文：[`dev.mysql.com/doc/refman/8.0/en/ipv6-local-connections.html`](https://dev.mysql.com/doc/refman/8.0/en/ipv6-local-connections.html)

#### 7.1.13.3 使用 IPv6 本地主机地址连接

以下步骤显示了如何配置 MySQL 以允许通过`::1`本地主机地址连接到本地服务器的客户端的 IPv6 连接。这里给出的说明假定您的系统支持 IPv6。

1.  使用适当的`bind_address`设置启动 MySQL 服务器，以允许其接受 IPv6 连接。例如，将以下行放入服务器选项文件并重新启动服务器：

    ```sql
    [mysqld]
    bind_address = *
    ```

    将*（或`::`）指定为`bind_address`的值允许在所有服务器主机 IPv4 和 IPv6 接口上进行 IPv4 和 IPv6 连接。如果您想将服务器绑定到特定地址列表，您可以在 MySQL 8.0.13 中通过为`bind_address`指定逗号分隔的值来实现。此示例指定了 IPv4 和 IPv6 的本地主机地址：

    ```sql
    [mysqld]
    bind_address = 127.0.0.1,::1
    ```

    有关更多信息，请参见第 7.1.8 节“服务器系统变量”中的`bind_address`描述。

1.  作为管理员，连接到服务器并为可以从`::1`本地 IPv6 主机地址连接的本地用户创建一个帐户：

    ```sql
    mysql> CREATE USER 'ipv6user'@'::1' IDENTIFIED BY 'ipv6pass';
    ```

    有关帐户名称中 IPv6 地址的允许语法，请参见第 8.2.4 节“指定帐户名称”。除了`CREATE USER`语句外，您还可以发出`GRANT`语句，为帐户授予特定权限，尽管这对本过程中的其余步骤并不是必需的。

1.  调用**mysql**客户端，使用新帐户连接到服务器：

    ```sql
    $> mysql -h ::1 -u ipv6user -pipv6pass
    ```

1.  尝试一些显示连接信息的简单语句：

    ```sql
    mysql> STATUS
    ...
    Connection:   ::1 via TCP/IP
    ...

    mysql> SELECT CURRENT_USER(), @@bind_address;
    +----------------+----------------+
    | CURRENT_USER() | @@bind_address |
    +----------------+----------------+
    | ipv6user@::1   | ::             |
    +----------------+----------------+
    ```
