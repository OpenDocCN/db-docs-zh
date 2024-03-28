# 7.1.13 IPv6 Support

> 原文：[`dev.mysql.com/doc/refman/8.0/en/ipv6-support.html`](https://dev.mysql.com/doc/refman/8.0/en/ipv6-support.html)

7.1.13.1 Verifying System Support for IPv6

7.1.13.2 Configuring the MySQL Server to Permit IPv6 Connections

7.1.13.3 Connecting Using the IPv6 Local Host Address

7.1.13.4 Connecting Using IPv6 Nonlocal Host Addresses

7.1.13.5 Obtaining an IPv6 Address from a Broker

MySQL 中对 IPv6 的支持包括以下功能：

+   MySQL 服务器可以接受通过 IPv6 连接的客户端的 TCP/IP 连接。���如，此命令通过 IPv6 连接到本地主机上的 MySQL 服务器：

    ```sql
    $> mysql -h ::1
    ```

    要使用此功能，必须满足两个条件：

    +   您的系统必须配置为支持 IPv6。参见 Section 7.1.13.1, “Verifying System Support for IPv6”。

    +   默认的 MySQL 服务器配置允许 IPv4 连接以及 IPv6 连接。要更改默认配置，请使用`bind_address`系统变量设置适当的值启动服务器。参见 Section 7.1.8, “Server System Variables”。

+   MySQL 帐户名称允许 IPv6 地址，以便 DBA 可以为通过 IPv6 连接到服务器的客户端指定权限。参见 Section 8.2.4, “Specifying Account Names”。IPv6 地址可以在语句中指定帐户名称，例如`CREATE USER`、`GRANT`和`REVOKE`。例如：

    ```sql
    mysql> CREATE USER 'bill'@'::1' IDENTIFIED BY 'secret';
    mysql> GRANT SELECT ON mydb.* TO 'bill'@'::1';
    ```

+   IPv6 函数允许在字符串和内部格式 IPv6 地址格式之间进行转换，并检查值是否表示有效的 IPv6 地址。例如，`INET6_ATON()`和`INET6_NTOA()`类似于`INET_ATON()`和`INET_NTOA()`，但除了 IPv4 地址外，还处理 IPv6 地址。参见 Section 14.23, “Miscellaneous Functions”。

+   从 MySQL 8.0.14 开始，Group Replication 组成员可以在组内使用 IPv6 地址进行通信。一个组可以包含使用 IPv6 和使用 IPv4 的成员的混合。参见 Section 20.5.5, “Support For IPv6 And For Mixed IPv6 And IPv4 Groups”。

以下各节描述了如何设置 MySQL，以便客户端可以通过 IPv6 连接到服务器。
