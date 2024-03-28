> 原文：[`dev.mysql.com/doc/refman/8.0/en/change-replication-source-to.html`](https://dev.mysql.com/doc/refman/8.0/en/change-replication-source-to.html)

#### 15.4.2.3 CHANGE REPLICATION SOURCE TO 语句

```sql
CHANGE REPLICATION SOURCE TO *option* [, *option*] ... [ *channel_option* ]

*option*: {
    SOURCE_BIND = '*interface_name*'
  | SOURCE_HOST = '*host_name*'
  | SOURCE_USER = '*user_name*'
  | SOURCE_PASSWORD = '*password*'
  | SOURCE_PORT = *port_num*
  | PRIVILEGE_CHECKS_USER = {NULL | '*account*'}
  | REQUIRE_ROW_FORMAT = {0|1}
  | REQUIRE_TABLE_PRIMARY_KEY_CHECK = {STREAM | ON | OFF | GENERATE}
  | ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS = {OFF | LOCAL | *uuid*}
  | SOURCE_LOG_FILE = '*source_log_name*'
  | SOURCE_LOG_POS = *source_log_pos*
  | SOURCE_AUTO_POSITION = {0|1}
  | RELAY_LOG_FILE = '*relay_log_name*'
  | RELAY_LOG_POS = *relay_log_pos*
  | SOURCE_HEARTBEAT_PERIOD = *interval*
  | SOURCE_CONNECT_RETRY = *interval*
  | SOURCE_RETRY_COUNT = *count*
  | SOURCE_CONNECTION_AUTO_FAILOVER = {0|1}
  | SOURCE_DELAY = *interval*
  | SOURCE_COMPRESSION_ALGORITHMS = '*algorithm*[,*algorithm*][,*algorithm*]'
  | SOURCE_ZSTD_COMPRESSION_LEVEL = *level*
  | SOURCE_SSL = {0|1}
  | SOURCE_SSL_CA = '*ca_file_name*'
  | SOURCE_SSL_CAPATH = '*ca_directory_name*'
  | SOURCE_SSL_CERT = '*cert_file_name*'
  | SOURCE_SSL_CRL = '*crl_file_name*'
  | SOURCE_SSL_CRLPATH = '*crl_directory_name*'
  | SOURCE_SSL_KEY = '*key_file_name*'
  | SOURCE_SSL_CIPHER = '*cipher_list*'
  | SOURCE_SSL_VERIFY_SERVER_CERT = {0|1}
  | SOURCE_TLS_VERSION = '*protocol_list*'
  | SOURCE_TLS_CIPHERSUITES = '*ciphersuite_list*'
  | SOURCE_PUBLIC_KEY_PATH = '*key_file_name*'
  | GET_SOURCE_PUBLIC_KEY = {0|1}
  | NETWORK_NAMESPACE = '*namespace*'
  | IGNORE_SERVER_IDS = (*server_id_list*),
  | GTID_ONLY = {0|1}
}

*channel_option*:
    FOR CHANNEL *channel*

*server_id_list*:
    [*server_id* [, *server_id*] ... ]
```

[`CHANGE REPLICATION SOURCE TO`](https://dev.mysql.com/doc/refman/8.0/en/change-replication-source-to.html)更改副本服务器用于连接到源并从源读取数据的参数。它还更新了复制元数据存储库的内容（参见[Section 19.2.4, “Relay Log and Replication Metadata Repositories”](https://dev.mysql.com/doc/refman/8.0/en/replica-logs.html)）。在 MySQL 8.0.23 及更高版本中，请使用[`CHANGE REPLICATION SOURCE TO`](https://dev.mysql.com/doc/refman/8.0/en/change-replication-source-to.html)代替已弃用的[`CHANGE MASTER TO`](https://dev.mysql.com/doc/refman/8.0/en/change-master-to.html)语句。

[`CHANGE REPLICATION SOURCE TO`](https://dev.mysql.com/doc/refman/8.0/en/change-replication-source-to.html)需要[`REPLICATION_SLAVE_ADMIN`](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_replication-slave-admin)权限（或已弃用的[`SUPER`](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_super)权限）。

在[`CHANGE REPLICATION SOURCE TO`](https://dev.mysql.com/doc/refman/8.0/en/change-replication-source-to.html)语句中未指定的选项保留其值，除非在以下讨论中另有说明。因此，在大多数情况下，无需指定不更改的选项。

用于`SOURCE_HOST`和其他[`CHANGE REPLICATION SOURCE TO`](https://dev.mysql.com/doc/refman/8.0/en/change-replication-source-to.html)选项的值会检查换行符（`\n`或`0x0A`）。这些值中存在这些字符会导致语句失败并出现错误。

可选的`FOR CHANNEL *`channel`*`子句允许您指定语句适用于哪个复制通道。提供`FOR CHANNEL *`channel`*`子句将[`CHANGE REPLICATION SOURCE TO`](https://dev.mysql.com/doc/refman/8.0/en/change-replication-source-to.html)语句应用于特定的复制通道，并用于添加新通道或修改现有通道。例如，要添加一个名为`channel2`的新通道：

```sql
CHANGE REPLICATION SOURCE TO SOURCE_HOST=host1, SOURCE_PORT=3002 FOR CHANNEL 'channel2';
```

如果未命名子句且不存在额外通道，则[`CHANGE REPLICATION SOURCE TO`](https://dev.mysql.com/doc/refman/8.0/en/change-replication-source-to.html)语句适用于默认通道，其名称为空字符串（""）。当设置了多个复制通道时，每个[`CHANGE REPLICATION SOURCE TO`](https://dev.mysql.com/doc/refman/8.0/en/change-replication-source-to.html)语句必须使用`FOR CHANNEL *`channel`*`子句命名通道。有关更多信息，请参见[Section 19.2.2, “Replication Channels”](https://dev.mysql.com/doc/refman/8.0/en/replication-channels.html)。

对于`CHANGE REPLICATION SOURCE TO`语句的某些选项，您必须在发出`CHANGE REPLICATION SOURCE TO`语句之前发出`STOP REPLICA`语句（以及之后发出`START REPLICA`语句）。有时，您只需要停止复制 SQL（应用程序）线程或复制 I/O（接收）线程，而不是两者都要停止：

+   当应用程序线程停止时，您可以使用`RELAY_LOG_FILE`、`RELAY_LOG_POS`和`SOURCE_DELAY`选项的任何允许组合执行`CHANGE REPLICATION SOURCE TO`，即使复制接收线程正在运行。当接收线程正在运行时，不得使用此语句的其他选项。

+   当接收线程停止时，您可以使用此语句的任何选项（以任何允许的组合）执行`CHANGE REPLICATION SOURCE TO`，*除了* `RELAY_LOG_FILE`、`RELAY_LOG_POS`、`SOURCE_DELAY`或`SOURCE_AUTO_POSITION = 1`，即使应用程序线程正在运行。

+   在发出使用`SOURCE_AUTO_POSITION = 1`、`GTID_ONLY = 1`或`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`的`CHANGE REPLICATION SOURCE TO`语句之前，必须停止接收线程和应用程序线程。

您可以使用`SHOW REPLICA STATUS`检查复制应用程序线程和复制接收线程的当前状态。请注意，Group Replication 应用程序通道（`group_replication_applier`）没有接收线程，只有一个应用程序线程。

`CHANGE REPLICATION SOURCE TO`语句具有许多您在执行之前应该了解的副作用和交互作用：

+   `CHANGE REPLICATION SOURCE TO`会导致正在进行的事务隐式提交。请参阅第 15.3.3 节，“导致隐式提交的语句”。

+   `CHANGE REPLICATION SOURCE TO`会将`SOURCE_HOST`、`SOURCE_PORT`、`SOURCE_LOG_FILE`和`SOURCE_LOG_POS`的先前值写入错误日志，以及在执行之前有关副本状态的其他信息。

+   如果您正在使用基于语句的复制和临时表，可能会出现在`CHANGE REPLICATION SOURCE TO`语句之后的`STOP REPLICA`语句在副本上留下临时表的情况。每当发生这种情况时，都会发出警告(`ER_WARN_OPEN_TEMP_TABLES_MUST_BE_ZERO`)。在这种情况下，您可以通过确保在执行`CHANGE REPLICATION SOURCE TO`语句之前，`Replica_open_temp_tables`或`Slave_open_temp_tables`系统状态变量的值等于 0 来避免这种情况。

+   当使用多线程副本（`replica_parallel_workers` > 0）时，停止副本可能导致已从中继日志执行的事务序列中存在间隙，无论副本是有意停止还是其他情况。当存在这样的间隙时，发出`CHANGE REPLICATION SOURCE TO`将失败。在这种情况下的解决方案是发出`START REPLICA UNTIL SQL_AFTER_MTS_GAPS`，以确保关闭这些间隙。从 MySQL 8.0.26 开始，当使用基于 GTID 的复制和 GTID 自动定位时，完全跳过检查事务序列中的间隙的过程，因为可以使用 GTID 自动定位解决事务间隙。在这种情况下，仍然可以使用`CHANGE REPLICATION SOURCE TO`。

下列选项适用于`CHANGE REPLICATION SOURCE TO`语句：

+   `ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS = {OFF | LOCAL | *`uuid`*}`

    使复制通道为没有 GTID 的复制事务分配一个 GTID，从不使用基于 GTID 的复制的源复制到使用基于 GTID 的副本。对于多源副本，您可以同时使用`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`和不使用该选项的通道。默认值为`OFF`，表示不使用该功能。

    `LOCAL`分配一个包括副本自身 UUID（`server_uuid`设置）的 GTID。`*`uuid`*`分配一个包括指定 UUID 的 GTID，例如复制源服务器的`server_uuid`设置。使用非本地 UUID 可以区分在副本上发起的事务和在源上发起的事务，对于多源副本，还可以区分在不同源上发起的事务。您选择的 UUID 仅对副本自身使用有意义。如果源发送的任何事务已经具有 GTID，则保留该 GTID。

    专用于组复制的通道不能使用`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`，但是在作为组复制组成员的服务器实例上的另一个源的异步复制通道可以这样做。在这种情况下，不要将组复制组名称指定为创建 GTID 的 UUID。

    要将`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`设置为`LOCAL`或`*`uuid`*`，副本必须设置`gtid_mode=ON`，并且之后不能更改。此选项用于与具有基于二进制日志文件位置的复制的源一起使用，因此不能为通道设置`SOURCE_AUTO_POSITION=1`。在设置此选项之前，必须停止复制 SQL 线程和复制 I/O（接收器）线程。

    重要提示

    使用任何通道上的`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`设置的副本不能在需要故障转移时晋升为替换复制源服务器，并且不能使用从副本中获取的备份来恢复复制源服务器。相同的限制也适用于替换或恢复其他使用任何通道上的`ASSIGN_GTIDS_TO_ANONYMOUS_TRANSACTIONS`的副本。

    有关更多限制和信息，请参阅第 19.1.3.6 节，“从没有 GTID 的源复制到具有 GTID 的副本”。

+   `GET_SOURCE_PUBLIC_KEY = {0|1}`

    通过请求源的公钥启用基于 RSA 密钥对的密码交换。此选项默认禁用。

    此选项适用于使用`caching_sha2_password`认证插件进行身份验证的复制品。对于使用此插件进行身份验证的帐户的连接，源不会发送公钥，除非请求，因此必须请求或在客户端中指定。如果给出`SOURCE_PUBLIC_KEY_PATH`并指定有效的公钥文件，则它优先于`GET_SOURCE_PUBLIC_KEY`。如果您正在使用使用`caching_sha2_password`插件进行身份验证的复制用户帐户（这是从 MySQL 8.0 开始的默认设置），并且您没有使用安全连接，则必须指定此选项或`SOURCE_PUBLIC_KEY_PATH`选项以向复制品提供 RSA 公钥。

+   `GTID_ONLY = {0|1}`

    停止复制通道在复制元数据存储库中持久化文件名和文件位置。`GTID_ONLY`从 MySQL 8.0.27 版本开始提供。`GTID_ONLY`选项在异步复制通道中默认禁用，但在组复制通道中默认启用，并且无法禁用。

    对于具有此设置的复制通道，仍会跟踪内存中的文件位置，并且出于调试目的，文件位置仍然可以在错误消息和通过诸如`SHOW REPLICA STATUS`语句（如果它们过时，则显示为无效）等接口中观察到。但是，在 GTID 基础复制实际上不需要它们的情况下，包括事务排队和应用程序过程中，避免了持久化和检查文件位置所需的写入和读取。

    只有在停止复制 SQL（应用程序）线程和复制 I/O（接收器）线程时才能使用此选项。要为复制通道设置`GTID_ONLY = 1`，必须在服务器上使用 GTID（`gtid_mode = ON`），并且源上必须使用基于行的二进制日志记录（不支持基于语句的复制）。必须为复制通道设置选项`REQUIRE_ROW_FORMAT = 1`和`SOURCE_AUTO_POSITION = 1`。

    当设置`GTID_ONLY = 1`时，如果服务器上的系统变量`replica_parallel_workers`设置为零，则复制品会使用`replica_parallel_workers=1`，因此它在技术上始终是一个多线程应用程序。这是因为多线程应用程序使用保存的位置而不是复制元数据存储库来定位需要重新应用的事务的起始位置。

    如果在设置`GTID_ONLY`后禁用它，现有的中继日志将被删除，并且现有的已知的二进制日志文件位置将被保留，即使它们已经过时。在复制元数据存储库中，二进制日志和中继日志的文件位置可能是无效的，如果是这种情况，则会返回警告。只要`SOURCE_AUTO_POSITION`仍然启用，GTID 自动定位将被用来提供正确的定位。

    如果你也禁用了`SOURCE_AUTO_POSITION`，则在复制元数据存储库中使用二进制日志和中继日志的文件位置进行定位，如果它们是有效的。如果它们被标记为无效，你必须提供一个有效的二进制日志文件名和位置（`SOURCE_LOG_FILE`和`SOURCE_LOG_POS`）。如果你还提供了一个中继日志文件名和位置（`RELAY_LOG_FILE`和`RELAY_LOG_POS`），则中继日志将被保留，而应用程序位置将被设置为指定位置。 GTID 自动跳过确保任何已经应用的事务都会被跳过，即使最终的应用程序位置不正确。

+   `IGNORE_SERVER_IDS = (*`server_id_list`*)`

    使复制忽略来自指定服务器的事件。该选项接受一个逗号分隔的 0 个或多个服务器 ID 的列表。来自服务器的日志旋转和删除事件不会被忽略，并且会记录在中继日志中。

    在循环复制中，原始服务器通常充当其自己事件的终结者，以便它们不会被应用多次。因此，在循环复制中，当圈中的一个服务器被移除时，这个选项是有用的。假设你有一个带有服务器 ID 为 1、2、3 和 4 的 4 台服务器的循环复制设置，并且服务器 3 失败了。在通过从服务器 2 开始到服务器 4 启动复制来填补这个空白时，你可以在服务器 4 上发出的`CHANGE REPLICATION SOURCE TO`语句中包含`IGNORE_SERVER_IDS = (3)`，告诉��使用服务器 2 作为其源，而不是服务器 3。这样做会导致它忽略并不传播任何源自不再使用的服务器的语句。

    如果`IGNORE_SERVER_IDS`包含服务器自己的 ID，并且服务器是使用`--replicate-same-server-id`选项启动的，则会导致错误。

    注意

    当全局事务标识符（GTID）用于复制时，已经应用的事务会自动被忽略，因此不需要使用`IGNORE_SERVER_IDS`函数，该函数已被弃用。如果为服务器设置了`gtid_mode=ON`，则如果在`CHANGE REPLICATION SOURCE TO`语句中包含`IGNORE_SERVER_IDS`选项，则会发出弃用警告。

    源元数据存储库和`SHOW REPLICA STATUS`的输出提供当前被忽略的服务器列表。有关更多信息，请参见第 19.2.4.2 节，“复制元数据存储库”和第 15.7.7.35 节，“显示 REPLICA 状态语句”。

    如果发出不带任何`IGNORE_SERVER_IDS`选项的`CHANGE REPLICATION SOURCE TO`语句，则会保留任何现有列表。要清除被忽略服务器的列表，必须使用带有空列表的选项：

    ```sql
    CHANGE REPLICATION SOURCE TO IGNORE_SERVER_IDS = ();
    ```

    `RESET REPLICA ALL`清除`IGNORE_SERVER_IDS`。

    注意

    如果在任何通道存在使用`IGNORE_SERVER_IDS`设置的现有服务器 ID 时发出`SET GTID_MODE=ON`，则会发出弃用警告。在启动基于 GTID 的复制之前，请检查并清除涉及服务器上的所有被忽略的服务器 ID 列表。`SHOW REPLICA STATUS`语句显示被忽略的 ID 列表（如果有）。如果收到弃用警告，仍然可以通过发出包含带有空列表的`IGNORE_SERVER_IDS`选项的`CHANGE REPLICATION SOURCE TO`语句来清除列表。

+   `NETWORK_NAMESPACE = '*`namespace`*'`

    用于 TCP/IP 连接到复制源服务器的网络命名空间，或者如果使用 MySQL 通信堆栈，则用于 Group Replication 的组通信连接。字符串值的最大长度为 64 个字符。如果省略此选项，则副本的连接将使用默认（全局）命名空间。在不实现网络命名空间支持的平台上，当副本尝试连接到源时会发生故障。有关网络命名空间的信息，请参见第 7.1.14 节，“网络命名空间支持”。`NETWORK_NAMESPACE`在 MySQL 8.0.22 中可用。

+   `PRIVILEGE_CHECKS_USER = {NULL | '*`account`*'}`

    为指定通道提供安全上下文的用户帐户名称。`NULL`是默认值，表示不使用安全上下文。`PRIVILEGE_CHECKS_USER`在 MySQL 8.0.18 中可用。

    用户帐户的用户名和主机名必须遵循 第 8.2.4 节“指定帐户名称” 中描述的语法，并且用户不能是匿名用户（具有空白用户名）或 `CURRENT_USER`。帐户必须具有 `REPLICATION_APPLIER` 权限，以及执行在通道上复制的事务所需的权限。有关帐户所需权限的详细信息，请参见 第 19.3.3 节“复制权限检查”。当重新启动复制通道时，权限检查将从那一点开始应用。如果您不指定通道且没有其他通道存在，则该语句将应用于默认通道。

    当设置了 `PRIVILEGE_CHECKS_USER` 时，强烈建议使用基于行的二进制日志记录，并且您可以设置 `REQUIRE_ROW_FORMAT` 来强制执行此操作。例如，要在运行中的副本上启动通道 `channel_1` 上的权限检查，请发出以下语句：

    ```sql
    STOP REPLICA FOR CHANNEL 'channel_1';

    CHANGE REPLICATION SOURCE TO
        PRIVILEGE_CHECKS_USER = '*user*'@'*host*',
        REQUIRE_ROW_FORMAT = 1,
        FOR CHANNEL 'channel_1';

    START REPLICA FOR CHANNEL 'channel_1';
    ```

+   `RELAY_LOG_FILE = '*`relay_log_file`*'`，`RELAY_LOG_POS = '*`relay_log_pos`*'`

    中继日志文件名及文件中的位置，在该位置，复制 SQL 线程在下次启动时开始从副本的中继日志中读取的位置。`RELAY_LOG_FILE` 可以使用绝对路径或相对路径，并且使用与 `SOURCE_LOG_FILE` 相同的基本名称。字符串值的最大长度为 511 个字符。

    当复制 SQL（应用程序）线程停止时，可以在运行中的副本上执行一个使用 `RELAY_LOG_FILE`、`RELAY_LOG_POS` 或两者选项的 `CHANGE REPLICATION SOURCE TO` 语句。如果至少有一个复制应用程序线程和复制 I/O（接收器）线程正在运行，则中继日志将被保留。如果两个线程都停止，则除非至少指定一个 `RELAY_LOG_FILE` 或 `RELAY_LOG_POS`，否则所有中继日志文件都将被删除。对于仅具有应用程序线程而没有接收器线程的 Group Replication 应用程序通道（`group_replication_applier`），如果应用程序线程停止，但是对于该通道，您不能使用 `RELAY_LOG_FILE` 和 `RELAY_LOG_POS` 选项。

+   `REQUIRE_ROW_FORMAT = {0|1}`

    仅允许处理基于行的复制事件的复制通道。此选项阻止复制应用程序执行诸如创建临时表和执行`LOAD DATA INFILE`请求等操作，从而增加了通道的安全性。对于异步复制通道，默认情况下禁用`REQUIRE_ROW_FORMAT`选项，但对于组复制通道，默认情况下启用，并且无法禁用。有关更多信息，请参见第 19.3.3 节，“复制权限检查”。`REQUIRE_ROW_FORMAT`从 MySQL 8.0.19 开始提供。

+   `REQUIRE_TABLE_PRIMARY_KEY_CHECK = {STREAM | ON | OFF | GENERATE}`

    从 MySQL 8.0.20 开始提供，此选项允许复制品设置自己的主键检查策略，如下：

    +   `ON`：复制品设置`sql_require_primary_key = ON`；任何复制的`CREATE TABLE`或`ALTER TABLE`语句必须生成包含主键的表。

    +   `OFF`：复制品设置`sql_require_primary_key = OFF`；不会检查任何复制的`CREATE TABLE`或`ALTER TABLE`语句是否存在主键。

    +   `STREAM`：复制品使用从源复制的`sql_require_primary_key`的任何值来处理每个事务。这是默认值和默认行为。

    +   `GENERATE`：在 MySQL 8.0.32 中添加，这会导致复制品为任何缺少主键的`InnoDB`表生成一个不可见的主键。有关更多信息，请参见第 15.1.20.11 节，“生成的不可见主键”。

        `GENERATE`与组复制不兼容；您可以使用`ON`、`OFF`或`STREAM`。

    基于源或复制品表上仅存在生成的不可见主键的差异得到 MySQL 复制支持，只要源支持 GIPKs（MySQL 8.0.30 及更高版本），而复制品使用 MySQL 版本 8.0.32 或更高版本。如果在复制品上使用 GIPKs 并从使用 MySQL 8.0.29 或更早版本的源进行复制，则应注意，在这种情况下，除了复制品上的额外 GIPK 之外，不支持模式中的这种差异，并且可能导致复制错误。

    对于多源复制，将`REQUIRE_TABLE_PRIMARY_KEY_CHECK`设置为`ON`或`OFF`可以使副本在不同源的复制通道之间规范化行为，并为`sql_require_primary_key`保持一致的设置。使用`ON`可以防止在多个源更新相同的表集时意外丢失主键。使用`OFF`可以让可以操作主键的源与不能操作主键的源一起工作。

    在多个副本的情况下，当`REQUIRE_TABLE_PRIMARY_KEY_CHECK`设置为`GENERATE`时，给定副本添加的生成的不可见主键与其他副本上添加的任何此类键是独立的。这意味着，如果使用生成的不可见主键，不同副本上生成的主键列中的值不能保证相同。当故障转移到这样的副本时，这可能是一个问题。

    当`PRIVILEGE_CHECKS_USER`为`NULL`（默认值）时，用户帐户不需要管理级别权限来设置受限制的会话变量。将此选项设置为`NULL`以外的值意味着，当`REQUIRE_TABLE_PRIMARY_KEY_CHECK`为`ON`，`OFF`或`GENERATE`时，用户帐户不需要会话管理级别权限来设置受限制的会话变量，如`sql_require_primary_key`，避免需要授予帐户此类权限。有关更多信息，请参见第 19.3.3 节，“复制权限检查”。

+   `SOURCE_AUTO_POSITION = {0|1}`

    使副本尝试使用基于 GTID 的复制的自动定位功能连接到源，而不是基于二进制日志文件的位置。此选项用于启动使用基于 GTID 的复制的副本。默认值为 0，表示不使用 GTID 自动定位和基于 GTID 的复制。只有在停止复制 SQL（应用程序）线程和复制 I/O（接收器）线程时，才能将此选项与`CHANGE REPLICATION SOURCE TO`一起使用。

    副本和源都必须启用 GTIDs（`GTID_MODE=ON`，在副本上为`ON_PERMISSIVE`或`OFF_PERMISSIVE`，在源上为`GTID_MODE=ON`）。`SOURCE_LOG_FILE`，`SOURCE_LOG_POS`，`RELAY_LOG_FILE`和`RELAY_LOG_POS`不能与`SOURCE_AUTO_POSITION = 1`一起指定。如果在副本上启用了多源复制，则需要为每个适用的复制通道设置`SOURCE_AUTO_POSITION = 1`选项。

    设置`SOURCE_AUTO_POSITION = 1`后，在初始连接握手中，副本发送一个包含其已接收、已提交或两者都包含的事务的 GTID 集。源端通过发送其二进制日志中所有 GTID 不包含在副本发送的 GTID 集中的事务来做出响应。这种交换确保源端只发送副本尚未记录或提交的具有 GTID 的事务。如果副本从多个源接收事务，如钻石拓扑结构的情况，自动跳过功能确保事务不会被应用两次。有关副本发送的 GTID 集如何计算的详细信息，请参见第 19.1.3.3 节，“GTID 自动定位”。

    如果应该由源端发送的任何事务已从源端的二进制日志中清除，或者通过其他方法添加到`gtid_purged`系统变量的 GTID 集中，源端将向副本发送错误`ER_MASTER_HAS_PURGED_REQUIRED_GTIDS`，并且复制不会启动。缺失的已清除事务的 GTID 将在源端的错误日志中被识别并列在警告消息`ER_FOUND_MISSING_GTIDS`中。此外，如果在事务交换过程中发现副本已记录或提交具有源端 UUID 的 GTID 的事务，但源端本身尚未提交它们，则源端将向副本发送错误`ER_SLAVE_HAS_MORE_GTIDS_THAN_MASTER`，并且复制不会启动。有关如何处理这些情况的信息，请参见第 19.1.3.3 节，“GTID 自动定位”。

    您可以通过检查性能模式`replication_connection_status`表或`SHOW REPLICA STATUS`语句的输出来查看是否启用了 GTID 自动定位的复制。再次禁用`SOURCE_AUTO_POSITION`选项会使副本恢复到基于文件的复制。

+   `SOURCE_BIND = '*`interface_name`*'`

    确定副本的哪个网络接口被选择用于连接到源端，适用于具有多个网络接口的副本。指定网络接口的 IP 地址。字符串值的最大长度为 255 个字符。

    配置此选项的 IP 地址（如果有）可以在`SHOW REPLICA STATUS`输出的`Source_Bind`列中看到。在源元数据存储库表`mysql.slave_master_info`中，该值可以在`Source_bind`列中看到。将副本绑定到特定网络接口的能力也受到 NDB Cluster 的支持。

+   [`SOURCE_COMPRESSION_ALGORITHMS = '*`algorithm`*[,*`algorithm`*][,*`algorithm`*]'`](change-replication-source-to.html#crs-opt-source_compression_algorithms)

    指定连接到复制源服务器的允许压缩算法之一、两个或三个，用逗号分隔。字符串值的最大长度为 99 个字符。默认值为`uncompressed`。

    可用的算法是`zlib`、`zstd`和`uncompressed`，与`protocol_compression_algorithms`系统变量相同。算法可以以任何顺序指定，但这不是优先顺序 - 算法协商过程尝试使用`zlib`，然后`zstd`，然后`uncompressed`，如果它们被指定。`SOURCE_COMPRESSION_ALGORITHMS`自 MySQL 8.0.18 起可用。

    `SOURCE_COMPRESSION_ALGORITHMS`的值仅在禁用`replica_compressed_protocol`或`slave_compressed_protocol`系统变量时适用。如果启用了`replica_compressed_protocol`或`slave_compressed_protocol`，它优先于`SOURCE_COMPRESSION_ALGORITHMS`，并且如果源和副本都支持该算法，则连接到源使用`zlib`压缩。有关更多信息，请参见第 6.2.8 节，“连接压缩控制”。

    二进制日志事务压缩（自 MySQL 8.0.20 起可用），由`binlog_transaction_compression`系统变量激活，也可用于节省带宽。如果与连接压缩结合使用，连接压缩的机会减少，但仍可压缩标头以及未压缩的事件和事务有效载荷。有关二进制日志事务压缩的更多信息，请参见第 7.4.4.5 节，“二进制日志事务压缩”。

+   `SOURCE_CONNECT_RETRY = *`interval`*`

    指定副本在连接到源超时后重新连接尝试之间的秒数间隔。默认间隔为 60。

    尝试次数受`SOURCE_RETRY_COUNT`选项限制。如果使用默认设置，副本在重新连接尝试之间等待 60 秒（`SOURCE_CONNECT_RETRY=60`），并以这个速率持续尝试重新连接 60 天（`SOURCE_RETRY_COUNT=86400`）。这些值记录在源元数据存储库中，并显示在`replication_connection_configuration`性能模式表中。

+   `SOURCE_CONNECTION_AUTO_FAILOVER = {0|1}`

    如果有一个或多个备用复制源服务器可用（即有多个共享复制数据的 MySQL 服务器或服务器组），则为复制通道激活异步连接故障转移机制。`SOURCE_CONNECTION_AUTO_FAILOVER`从 MySQL 8.0.22 版本开始提供。默认值为 0，表示未激活该机制。有关完整信息和设置此功能的说明，请参阅第 19.4.9.2 节，“副本的异步连接故障转移”。

    在由`SOURCE_CONNECT_RETRY`和`SOURCE_RETRY_COUNT`控制的重新连接尝试耗尽后，异步连接故障转移机制接管。它将副本重新连接到从指定源列表中选择的备用源，您可以使用`asynchronous_connection_failover_add_source`和`asynchronous_connection_failover_delete_source`函数进行管理。要添加和删除受管理的服务器组，请改用`asynchronous_connection_failover_add_managed`和`asynchronous_connection_failover_delete_managed`函数。有关更多信息，请参阅第 19.4.9 节，“使用异步连接故障转移切换源和副本”。

    重要提示

    1.  只有在使用 GTID 自动定位时（`SOURCE_AUTO_POSITION = 1`）才能设置`SOURCE_CONNECTION_AUTO_FAILOVER = 1`。

    1.  当你设置 `SOURCE_CONNECTION_AUTO_FAILOVER = 1` 时，将 `SOURCE_RETRY_COUNT` 和 `SOURCE_CONNECT_RETRY` 设置为最小值，只允许几次重试尝试与相同源进行连接，以防连接失败是由瞬时网络中断引起的。否则，异步连接故障转移机制无法及时激活。适当的值是 `SOURCE_RETRY_COUNT=3` 和 `SOURCE_CONNECT_RETRY=10`，这使得副本在每次连接尝试之间间隔 10 秒的情况下重试连接 3 次。

    1.  当你设置 `SOURCE_CONNECTION_AUTO_FAILOVER = 1` 时，复制元数据存储库必须包含用于连接到复制通道源列表中所有服务器的复制用户帐户的凭据。该帐户还必须对性能模式表具有 `SELECT` 权限。这些凭据可以使用带有 `SOURCE_USER` 和 `SOURCE_PASSWORD` 选项的 `CHANGE REPLICATION SOURCE TO` 语句进行设置。有关更多信息，请参见 第 19.4.9 节，“使用异步连接故障转移切换源和副本”。

    1.  从 MySQL 8.0.27 开始，当你设置 `SOURCE_CONNECTION_AUTO_FAILOVER = 1` 时，如果这个复制通道在单主模式下的组复制主节点上，副本的异步连接故障转移将自动激活。启用此功能后，如果正在复制的主节点下线或进入错误状态，新的主节点在选举时将在同一通道上开始复制。如果要使用此功能，这个复制通道还必须在复制组中的所有辅助服务器上设置，并在任何新加入的成员上设置。（如果使用 MySQL 的克隆功能进行服务器配置，则所有这些都会自动发生。）如果不想使用此功能，请使用 `group_replication_disable_member_action()` 函数禁用默认启用的 Group Replication 成员操作 `mysql_start_failover_channels_if_primary`。有关更多信息，请参见 第 19.4.9.2 节，“副本的异步连接故障转移”。

+   `SOURCE_DELAY = *`interval`*`

    指定副本必须滞后源多少秒。从源接收的事件直到至少延迟*`interval`*秒后才执行。*`interval`*必须是 0 到 2³¹−1 范围内的非负整数。默认值为 0。更多信息，请参见 Section 19.4.11, “Delayed Replication”。

    使用`SOURCE_DELAY`选项的`CHANGE REPLICATION SOURCE TO`语句可以在运行中的副本上执行，当复制 SQL 线程停止时。

+   `SOURCE_HEARTBEAT_PERIOD = *`interval`*`

    控制心跳间隔，防止在没有数据的情况下发生连接超时，如果连接仍然良好。在那么多秒后向副本发送心跳信号，并且每当源的二进制日志更新为一个事件时，等待时间就会重置。因此，只有在二进制日志文件中有一段时间没有发送的事件时，源才会发送心跳。

    心跳间隔*`interval`*是一个十进制值，范围为 0 到 4294967 秒，分辨率为毫秒；最小的非零值为 0.001。将*`interval`*设置为 0 会完全禁用心跳。心跳间隔默认为`replica_net_timeout`或`slave_net_timeout`系统变量值的一半。它记录在源元数据存储库中，并显示在`replication_connection_configuration`性能模式表中。

    系统变量`replica_net_timeout`（从 MySQL 8.0.26 开始）或`slave_net_timeout`（MySQL 8.0.26 之前）指定副本等待来自源的更多数据或心跳信号的秒数，超过该时间副本将认为连接已中断，中止读取并尝试重新连接。默认值为 60 秒（一分钟）。请注意，对`replica_net_timeout`或`slave_net_timeout`的值或默认设置的更改不会自动更改心跳间隔，无论是明确设置还是使用先前计算的默认值。如果将全局值`replica_net_timeout`或`slave_net_timeout`设置为小于当前心跳间隔的值，将发出警告。如果更改了`replica_net_timeout`或`slave_net_timeout`，还必须发出`CHANGE REPLICATION SOURCE TO`以将心跳间隔调整为适当的值，以便心跳信号在连接超时之前发生。如果不这样做，心跳信号将不起作用，如果未从源接收到数据，则副本可能会进行重复的重新连接尝试，从而创建僵尸转储线程。

+   `SOURCE_HOST = '*`host_name`*'`

    复制源服务器的主机名或 IP 地址。副本使用此信息连接到源。字符串值的最大长度为 255 个字符。

    如果您指定了`SOURCE_HOST`或`SOURCE_PORT`，则副本会假定源服务器与之前不同（即使选项值与当前值相同）。在这种情况下，源的二进制日志文件名和位置的旧值被认为不再适用，因此如果您在语句中未指定`SOURCE_LOG_FILE`和`SOURCE_LOG_POS`，则会自动追加`SOURCE_LOG_FILE=''`和`SOURCE_LOG_POS=4`。

    将`SOURCE_HOST=''`设置为（即，将其值明确设置为空字符串）*不*等同于根本不设置`SOURCE_HOST`。尝试将`SOURCE_HOST`设置为空字符串会导致错误。

+   `SOURCE_LOG_FILE = '*`source_log_name`*'`, `SOURCE_LOG_POS = *`source_log_pos`*`

    二进制日志文件名以及在该文件中的位置，复制 I/O（接收器）线程在下次启动时从源二进制日志中开始读取的位置。如果使用基于二进制日志文件位置的复制，请指定这些选项。

    `SOURCE_LOG_FILE`必须包括源服务器上可用的特定二进制日志文件的数字后缀，例如，`SOURCE_LOG_FILE='binlog.000145'`。字符串值的最大长度为 511 个字符。

    `SOURCE_LOG_POS`是副本在该文件中开始读取的数字位置。`SOURCE_LOG_POS=4`表示二进制日志文件中事件的开始。

    如果指定了`SOURCE_LOG_FILE`或`SOURCE_LOG_POS`中的任一个，则不能指定`SOURCE_AUTO_POSITION = 1`，该选项用于基于 GTID 的复制。

    如果未指定`SOURCE_LOG_FILE`或`SOURCE_LOG_POS`中的任何一个，则副本使用*复制 SQL 线程*在发出`CHANGE REPLICATION SOURCE TO`之前的最后坐标。这确保了即使复制 SQL（应用程序）线程比复制 I/O（接收器）线程晚，复制也不会出现不连续。

+   `SOURCE_PASSWORD = '*`password`*'`

    用于连接到复制源服务器的复制用户帐户的密码。字符串值的最大长度为 32 个字符。如果指定了`SOURCE_PASSWORD`，则还需要`SOURCE_USER`。

    在`CHANGE REPLICATION SOURCE TO`语句中用于复制用户帐户的密码长度限制为 32 个字符。尝试使用超过 32 个字符的密码会导致`CHANGE REPLICATION SOURCE TO`失败。

    密码在 MySQL Server 的日志、性能模式表和`SHOW PROCESSLIST`语句中被掩盖。

+   `SOURCE_PORT = *`port_num`*`

    副本用于连接到复制源服务器的 TCP/IP 端口号。

    注意

    复制不能使用 Unix 套接字文件。您必须能够使用 TCP/IP 连接到复制源服务器。

    如果指定了`SOURCE_HOST`或`SOURCE_PORT`，则副本会假定源服务器与以前不同（即使选项值与当前值相同）。在这种情况下，源二进制日志文件名和位置的旧值被认为不再适用，因此如果在语句中不指定`SOURCE_LOG_FILE`和`SOURCE_LOG_POS`，则会自动附加`SOURCE_LOG_FILE=''`和`SOURCE_LOG_POS=4`。

+   `SOURCE_PUBLIC_KEY_PATH = '*`key_file_name`*'`

    通过提供包含源端所需的公钥副本的文件路径名，启用基于 RSA 密钥对的密码交换。文件必须采用 PEM 格式。字符串值的最大长度为 511 个字符。

    此选项适用于使用`sha256_password`或`caching_sha2_password`认证插件进行身份验证的副本。（对于`sha256_password`，只有在使用 OpenSSL 构建 MySQL 时才能使用`SOURCE_PUBLIC_KEY_PATH`。）如果您正在使用使用`caching_sha2_password`插件进行身份验证的复制用户帐户（这是 MySQL 8.0 的默认设置），并且您没有使用安全连接，则必须指定此选项或`GET_SOURCE_PUBLIC_KEY=1`选项之一以向副本提供 RSA 公钥。

+   `SOURCE_RETRY_COUNT = *`count`*`

    设置副本在与源的连接超时后进行的重新连接尝试的最大次数，由`replica_net_timeout`或`slave_net_timeout`系统变量确定。如果副本确实需要重新连接，则第一次重试会在超时后立即发生。默认值为 86400 次尝试。

    尝试之间的间隔由`SOURCE_CONNECT_RETRY`选项指定。如果使用默认设置，副本在重新连接尝试之间等待 60 秒（`SOURCE_CONNECT_RETRY=60`），并以此速率继续尝试重新连接 60 天（`SOURCE_RETRY_COUNT=86400`）。将`SOURCE_RETRY_COUNT`设置为 0 意味着没有重新连接尝试次数限制，因此副本将无限尝试重新连接。

    `SOURCE_CONNECT_RETRY`和`SOURCE_RETRY_COUNT`的值记录在源元数据存储库中，并显示在`replication_connection_configuration`性能模式表中。`SOURCE_RETRY_COUNT`取代了`--master-retry-count`服务器启动选项。

+   `SOURCE_SSL = {0|1}`

    指定副本是否加密复制连接。默认值为 0，表示副本不加密复制连接。如果设置`SOURCE_SSL=1`，则可以使用`SOURCE_SSL_*`xxx`*`和`SOURCE_TLS_*`xxx`*`选项配置加密。

    为复制连接设置`SOURCE_SSL=1`，然后不设置更多的`SOURCE_SSL_*`xxx`*`选项相当于为客户端设置`--ssl-mode=REQUIRED`，如加密连接的命令选项中所述。使用`SOURCE_SSL=1`，只有在可以建立加密连接时连接尝试才会成功。复制连接不会退回到未加密连接，因此没有与复制的`--ssl-mode=PREFERRED`设置相对应的设置。如果设置`SOURCE_SSL=0`，则相当于`--ssl-mode=DISABLED`。

    重要

    为了帮助防止复杂的中间人攻击，副本验证服务器的身份非常重要。您可以指定额外的`SOURCE_SSL_*`xxx`*`选项对应于设置`--ssl-mode=VERIFY_CA`和`--ssl-mode=VERIFY_IDENTITY`，这比默认设置更好地帮助防止这种类型的攻击。使用这些设置，副本会检查服务器的证书是否有效，并检查副本正在使用的主机名是否与服务器证书中的标识匹配。要实施这些验证级别之一，您必须首先确保服务器的 CA 证书可靠地可用于副本，否则将导致可用性问题。因此，它们不是默认设置。

+   `SOURCE_SSL_*`xxx`*`, `SOURCE_TLS_*`xxx`*`

    指定副本如何使用加密和密码来保护复制连接。即使在没有 SSL 支持的编译副本上也可以更改这些选项。它们保存在源元数据存储库中，但如果副本没有启用 SSL 支持，则会被忽略。字符串值为`SOURCE_SSL_*`xxx`*`和`SOURCE_TLS_*`xxx`*`选项的最大长度为 511 个字符，例外是`SOURCE_TLS_CIPHERSUITES`，其长度为 4000 个字符。

    `SOURCE_SSL_*`xxx`*`和`SOURCE_TLS_*`xxx`*`选项执行与加密连接的命令选项中描述的`--ssl-*`xxx`*`和`--tls-*`xxx`*`客户端选项相同的功能。两组选项之间的对应关系，以及使用`SOURCE_SSL_*`xxx`*`和`SOURCE_TLS_*`xxx`*`选项建立安全连接的方法，在第 19.3.1 节，“设置复制使用加密连接”中有解释。

+   `SOURCE_USER = '*`user_name`*'`

    复制用户帐户的用户名，用于连接到复制源服务器。字符串值的最大长度为 96 个字符。

    对于组复制，此帐户必须存在于复制组的每个成员中。如果 XCom 通信堆栈用于组，则用于分布式恢复，如果 MySQL 通信堆栈用于组，则用于组通信连接。对于 MySQL 通信堆栈，帐户必须具有`GROUP_REPLICATION_STREAM`权限。

    可以通过指定`SOURCE_USER=''`来设置空用户名称，但不能使用空用户名称启动复制通道。在 MySQL 8.0.21 之前的版本中，仅在需要出于安全目的清除先前使用的凭据时才设置空的`SOURCE_USER`用户名称。不要在此之后使用通道，因为在这些版本中存在一个错误，如果从存储库中读取空用户名称（例如，在组复制通道的自动重启期间），则可能会替换默认用户名称。从 MySQL 8.0.21 开始，可以设置空的`SOURCE_USER`用户名称，并且可以在之后使用通道，如果始终使用`START REPLICA`语句或`START GROUP_REPLICATION`语句提供用户凭据以启动复制通道。这种方法意味着复制通道始终需要操作员干预才能重新启动，但用户凭据不会记录在复制元数据存储库中。

    重要提示

    要使用使用`caching_sha2_password`插件进行身份验证的复制用户帐户连接到源，您必须设置安全连接，如 Section 19.3.1, “Setting Up Replication to Use Encrypted Connections”中所述，或启用不加密的连接以支持使用 RSA 密钥对进行密码交换。`caching_sha2_password`身份验证插件是从 MySQL 8.0 开始的新用户的默认插件（请参阅 Section 8.4.1.2, “Caching SHA-2 Pluggable Authentication”）。如果您创建或使用用于复制的用户帐户使用此身份验证插件，并且未使用安全连接，则必须启用基于 RSA 密钥对的密码交换以成功连接。您可以使用此语句的`SOURCE_PUBLIC_KEY_PATH`选项或`GET_SOURCE_PUBLIC_KEY=1`选项来执行此操作。

+   `SOURCE_ZSTD_COMPRESSION_LEVEL = *`level`*`

    用于使用`zstd`压缩算法的连接到复制源服务器的压缩级别。`SOURCE_ZSTD_COMPRESSION_LEVEL`自 MySQL 8.0.18 起可用。允许的级别为 1 到 22，较大的值表示较高级别的压缩。默认级别为 3。

    压缩级别设置对不使用`zstd`压缩的连接没有影响。有关更多信息，请参见第 6.2.8 节，“连接压缩控制”。

##### 示例

`CHANGE REPLICATION SOURCE TO`对于在具有源快照并记录了快照时间对应的源二进制日志坐标的情况下设置副本非常有用。在将快照加载到副本以与源同步之后，您可以在副本上运行`CHANGE REPLICATION SOURCE TO SOURCE_LOG_FILE='*`log_name`*', SOURCE_LOG_POS=*`log_pos`*`来指定副本应开始读取源二进制日志的坐标。以下示例更改了副本使用的源服务器并建立了副本开始读取的源二进制日志坐标：

```sql
CHANGE REPLICATION SOURCE TO
  SOURCE_HOST='source2.example.com',
  SOURCE_USER='replication',
  SOURCE_PASSWORD='*password*',
  SOURCE_PORT=3306,
  SOURCE_LOG_FILE='source2-bin.001',
  SOURCE_LOG_POS=4,
  SOURCE_CONNECT_RETRY=10;
```

有关在故障切换期间将现有副本切换到新源的过程，请参见第 19.4.8 节，“故障切换期间切换源”。

当源和副本上使用 GTID 时，请指定 GTID 自动定位，而不是提供二进制日志文件位置，如下例所示。有关在新服务器或停止的服务器、在线服务器或其他副本上配置和启动基于 GTID 的复制的完整说明，请参见第 19.1.3 节，“具有全局事务标识符的复制”。

```sql
CHANGE REPLICATION SOURCE TO
  SOURCE_HOST='source3.example.com',
  SOURCE_USER='replication',
  SOURCE_PASSWORD='*password*',
  SOURCE_PORT=3306,
  SOURCE_AUTO_POSITION = 1,
  FOR CHANNEL "source_3";
```

在此示例中，使用了多源复制，并且`CHANGE REPLICATION SOURCE TO`语句应用于连接副本到指定主机的复制通道`"source_3"`。有关设置多源复制的指导，请参见第 19.1.5 节，“MySQL 多源复制”。

下一个示例显示了如何使副本应用您想要重复的中继日志文件中的事务。为此，源不需要可达。您可以使用`CHANGE REPLICATION SOURCE TO`来定位您希望副本开始重新应用事务的中继日志位置，然后启动 SQL 线程：

```sql
CHANGE REPLICATION SOURCE TO
  RELAY_LOG_FILE='replica-relay-bin.006',
  RELAY_LOG_POS=4025;
START REPLICA SQL_THREAD;
```

`更改复制源为`也可以用于跳过导致复制停止的二进制日志中的事务。执行此操作的适当方法取决于是否使用 GTID。有关使用`更改复制源为`或其他方法跳过事务的说明，请参见第 19.1.7.3 节，“跳过事务”。
