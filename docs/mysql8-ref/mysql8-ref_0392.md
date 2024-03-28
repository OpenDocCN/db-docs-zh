# 8.3.5 重用 SSL 会话

> 原文：[`dev.mysql.com/doc/refman/8.0/en/reusing-ssl-sessions.html`](https://dev.mysql.com/doc/refman/8.0/en/reusing-ssl-sessions.html)

截至 MySQL 8.0.29，MySQL 客户端程序可以选择恢复先前的 SSL 会话，前提是服务器在其运行时缓存中具有该会话。本节描述了有利于 SSL 会话重用的条件，用于管理和监控会话缓存的服务器变量，以及用于存储和重用会话数据的客户端命令行选项。

+   用于 SSL 会话重用的服务器端运行时配置和监控

+   用于 SSL 会话重用的客户端端配置

每次完整的 TLS 交换在计算和网络开销方面都可能很昂贵，如果使用 TLSv1.3 则开销较小。通过从已建立的会话中提取会话票证，然后在建立下一个连接时提交该票证，如果会话可以被重用，则总体成本会降低。例如，考虑具有可以打开多个连接并生成更快速度的网页的好处。

通常，在 SSL 会话可以被重用之前，必须满足以下条件：

+   服务器必须将其会话缓存保留在内存中。

+   服务器端会话缓存超时时间不能已过期。

+   每个客户端都必须维护一个活动会话的缓存并保持其安全。

C 应用程序可以利用 C API 功能来启用加密连接的会话重用（参见 SSL 会话重用）。

#### 用于 SSL 会话重用的服务器端运行时配置和监控

为了创建初始的 TLS 上下文，服务器使用启动时上下文相关系统变量的值。为了暴露上下文值，服务器还初始化了一组相应的状态变量。以下表格显示了定义服务器运行时会话缓存的系统变量以及暴露当前活动会话缓存值的相应状态变量。

**表 8.15 用于会话重用的系统和状态变量**

| 系统变量名 | 对应的状态变量名 |
| --- | --- |
| `ssl_session_cache_mode` | `Ssl_session_cache_mode` |
| `ssl_session_cache_timeout` | `Ssl_session_cache_timeout` |

注意

当`ssl_session_cache_mode`服务器变量的值为`ON`时，这是默认模式，`Ssl_session_cache_mode`状态变量的值为`SERVER`。

SSL 会话缓存变量适用于`mysql_main`和`mysql_admin` TLS 通道。它们的值也作为性能模式`tls_channel_status`表中的属性公开，以及任何其他活动 TLS 上下文的属性。

要在运行时重新配置 SSL 会话缓存，请使用以下步骤：

1.  将应更改为其新值的每个与缓存相关的系统变量设置为其新值。例如，将缓存超时值从默认值（300 秒）更改为 600 秒：

    ```sql
    mysql> SET GLOBAL ssl_session_cache_timeout = 600;
    ```

    每对系统和状态变量的成员可能由于重新配置过程的方式而暂时具有不同的值。

    ```sql
    mysql> SHOW VARIABLES LIKE 'ssl_session_cache_timeout';
    +---------------------------+-------+
    | Variable_name             | Value |
    +---------------------------+-------+
    | ssl_session_cache_timeout | 600   |
    +---------------------------+-------+
    1 row in set (0.00 sec)

    mysql> SHOW STATUS LIKE 'Ssl_session_cache_timeout';
    +---------------------------+-------+
    | Variable_name             | Value |
    +---------------------------+-------+
    | Ssl_session_cache_timeout | 300   |
    +---------------------------+-------+
    1 row in set (0.00 sec)
    ```

    有关设置变量值的其他信息，请参阅系统变量赋值。

1.  执行`ALTER INSTANCE RELOAD TLS`。此语句会根据缓存相关系统变量的当前值重新配置活动的 TLS 上下文。它还会将缓存相关状态变量设置为反映新活动缓存值。该语句需要`CONNECTION_ADMIN`权限。

    ```sql
    mysql> ALTER INSTANCE RELOAD TLS;
    Query OK, 0 rows affected (0.01 sec)

    mysql> SHOW VARIABLES LIKE 'ssl_session_cache_timeout';
    +---------------------------+-------+
    | Variable_name             | Value |
    +---------------------------+-------+
    | ssl_session_cache_timeout | 600   |
    +---------------------------+-------+
    1 row in set (0.00 sec)

    mysql> SHOW STATUS LIKE 'Ssl_session_cache_timeout';
    +---------------------------+-------+
    | Variable_name             | Value |
    +---------------------------+-------+
    | Ssl_session_cache_timeout | 600   |
    +---------------------------+-------+
    1 row in set (0.00 sec)
    ```

    在执行`ALTER INSTANCE RELOAD TLS`后建立的新连接使用新的 TLS 上下文。现有连接不受影响。

#### SSL 会话重用的客户端端配置

所有 MySQL 客户端程序都能够重用先前会话，用于与同一服务器建立的新加密连接，前提是您在原始连接仍处于活动状态时存储了会话数据。会话数据存储到文件中，当您再次调用客户端时，���会指定此文件。

要存储和重用 SSL 会话数据，请使用以下步骤：

1.  调用**mysql**以建立到运行 MySQL 8.0.29 或更高版本的服务器的加密连接。

1.  使用**ssl_session_data_print**命令指定一个文件路径，您可以在其中安全地存储当前活动会话数据。例如：

    ```sql
    mysql> ssl_session_data_print ~/private-dir/session.txt
    ```

    会话数据以以空终止的、PEM 编码的 ANSI 字符串形式获取。如果省略路径和文件名，则该字符串将打印到标准输出。

1.  从命令解释器的提示符中，调用任何 MySQL 客户端程序以建立到同一服务器的新加密连接。要重用会话数据，请指定`--ssl-session-data`命令行选项和文件参数。

    例如，使用**mysql**建立一个新连接：

    ```sql
    mysql -u admin -p --ssl-session-data=~/private-dir/session.txt
    ```

    然后使用**mysqlshow**客户端：

    ```sql
    mysqlshow -u admin -p --ssl-session-data=~/private-dir/session.txt
    Enter password: *****
    +--------------------+
    |     Databases      |
    +--------------------+
    | information_schema |
    | mysql              |
    | performance_schema |
    | sys                |
    | world              |
    +--------------------+
    ```

    在每个示例中，客户端尝试恢复原始会话，同时与同一服务器建立新连接。

    要确认**mysql**是否重用了会话，请查看`status`命令的输出。如果当前活动的**mysql**连接确实恢复了会话，则状态信息包括`SSL 会话重用: true`。

除了**mysql**和**mysqlshow**外，SSL 会话重用也适用于**mysqladmin**、**mysqlbinlog**、**mysqlcheck**、**mysqldump**、**mysqlimport**、**mysqlpump**、**mysqlslap**、**mysqltest**、**mysql_migrate_keyring**、**mysql_secure_installation**和**mysql_upgrade**。

几种情况可能会阻止成功检索会话数据。例如，如果会话未完全连接、不是 SSL 会话、服务器尚未发送会话数据或 SSL 会话无法重用。即使有正确存储的会话数据，服务器的会话缓存也可能超时。无论原因是什么，如果指定了`--ssl-session-data`，但会话无法重用，则默认情况下会返回错误。例如：

```sql
mysqlshow -u admin -p --ssl-session-data=~/private-dir/session.txt
Enter password: *****
ERROR:
--ssl-session-data specified but the session was not reused.
```

为了抑制错误消息，并通过在命令行上静默创建一个新会话来建立连接，需要指定`--ssl-session-data-continue-on-failed-reuse`，以及`--ssl-session-data`。如果服务器的缓存超时已过期，可以将会话数据再次存储到同一文件中。默认服务器缓存超时可以延长（参见 SSL 会话重用的服务器端运行时配置和监控）。
