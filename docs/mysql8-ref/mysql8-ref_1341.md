> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-howto-slaveinit.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-howto-slaveinit.html)

#### 19.1.2.7 在副本上设置源配置

要设置副本与源通信以进行复制，请使用必要的连接信息配置副本。为此，在副本上执行`CHANGE REPLICATION SOURCE TO`语句（从 MySQL 8.0.23 开始）或`CHANGE MASTER TO`语句（在 MySQL 8.0.23 之前），将选项值替换为与您系统相关的实际值：

```sql
mysql> CHANGE MASTER TO
 ->     MASTER_HOST='*source_host_name*',
 ->     MASTER_USER='*replication_user_name*',
 ->     MASTER_PASSWORD='*replication_password*',
 ->     MASTER_LOG_FILE='*recorded_log_file_name*',
 ->     MASTER_LOG_POS=*recorded_log_position*;

Or from MySQL 8.0.23:
mysql> CHANGE REPLICATION SOURCE TO
 ->     SOURCE_HOST='*source_host_name*',
 ->     SOURCE_USER='*replication_user_name*',
 ->     SOURCE_PASSWORD='*replication_password*',
 ->     SOURCE_LOG_FILE='*recorded_log_file_name*',
 ->     SOURCE_LOG_POS=*recorded_log_position*;
```

注意

复制不能使用 Unix 套接字文件。您必须能够使用 TCP/IP 连接到源 MySQL 服务器。

`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`语句还有其他选项。例如，可以使用 SSL 设置安全复制。有关选项的完整列表以及有关字符串值选项的最大允许长度的信息，请参阅第 15.4.2.1 节“CHANGE MASTER TO Statement”。

重要提示

如第 19.1.2.3 节“为复制创建用户”中所述，如果您未使用安全连接，并且`SOURCE_USER` | `MASTER_USER`选项中指定的用户帐户使用`caching_sha2_password`插件进行身份验证（从 MySQL 8.0 开始的默认设置），则必须在`CHANGE REPLICATION SOURCE TO` | `CHANGE MASTER TO`语句中指定`SOURCE_PUBLIC_KEY_PATH` | `MASTER_PUBLIC_KEY_PATH`或`GET_SOURCE_PUBLIC_KEY` | `GET_MASTER_PUBLIC_KEY`选项，以启用基于 RSA 密钥对的密码交换。
