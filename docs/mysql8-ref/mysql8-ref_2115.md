> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-tls-channel-status-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-tls-channel-status-table.html)

#### 29.12.21.9 The tls_channel_status Table

连接接口 TLS 属性在服务器启动时设置，并可以使用`ALTER INSTANCE RELOAD TLS`语句在运行时进行更新。请参见 Server-Side Runtime Configuration and Monitoring for Encrypted Connections。

`tls_channel_status`表（自 MySQL 8.0.21 起可用）提供有关连接接口 TLS 属性的信息：

```sql
mysql> SELECT * FROM performance_schema.tls_channel_status\G
*************************** 1\. row ***************************
 CHANNEL: mysql_main
PROPERTY: Enabled
   VALUE: Yes
*************************** 2\. row ***************************
 CHANNEL: mysql_main
PROPERTY: ssl_accept_renegotiates
   VALUE: 0
*************************** 3\. row ***************************
 CHANNEL: mysql_main
PROPERTY: Ssl_accepts
   VALUE: 2
...
*************************** 29\. row ***************************
 CHANNEL: mysql_admin
PROPERTY: Enabled
   VALUE: No
*************************** 30\. row ***************************
 CHANNEL: mysql_admin
PROPERTY: ssl_accept_renegotiates
   VALUE: 0
*************************** 31\. row ***************************
 CHANNEL: mysql_admin
PROPERTY: Ssl_accepts
   VALUE: 0
...
```

`tls_channel_status`表具有以下列：

+   `CHANNEL`

    适用于 TLS 属性行的连接接口的名称。`mysql_main`和`mysql_admin`分别是主连接接口和管理连接接口的通道名称。有关不同接口的信息，请参见 Section 7.1.12.1, “Connection Interfaces”。

+   `PROPERTY`

    TLS 属性名称。`Enabled`属性的行指示整体接口状态，其中接口及其状态分别在`CHANNEL`和`VALUE`列中命名。其他属性名称指示特定的 TLS 属性。这些通常对应于与 TLS 相关的状态变量的名称。

+   `VALUE`

    TLS 属性值。

由该表公开的属性不是固定的，而是取决于每个通道实现的仪表化。

对于每个通道，具有`PROPERTY`值为`Enabled`的行指示通道是否支持加密连接，其他通道行指示 TLS 上下文属性：

+   对于`mysql_main`，`Enabled`属性为`yes`或`no`，表示主接口是否支持加密连接。其他通道行显示主接口的 TLS 上下文属性。

    对于主接口，可以使用以下语句获取类似的状态信息：

    ```sql
    SHOW GLOBAL STATUS LIKE 'current_tls%';
    SHOW GLOBAL STATUS LIKE 'ssl%';
    ```

+   对于`mysql_admin`，如果管理接口未启用或已启用但不支持加密连接，则`Enabled`属性为`no`。如果接口已启用并支持加密连接，则`Enabled`为`yes`。

    当 `Enabled` 为 `yes` 时，如果为该接口配置了一些非默认的 TLS 参数值，则其他 `mysql_admin` 行仅指示管理接口 TLS 上下文的通道属性。 （如果任何 `admin_tls_*`xxx`*` 或 `admin_ssl_*`xxx`*` 系统变量设置为与其默认值不同的值，则是这种情况。）否则，管理接口使用与主接口相同的 TLS 上下文。

`tls_channel_status` 表没有索引。

`TRUNCATE TABLE` 不允许用于 `tls_channel_status` 表。
