# 29.12.9 性能模式连接属性表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-connection-attribute-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-connection-attribute-tables.html)

29.12.9.1 session_account_connect_attrs 表

29.12.9.2 session_connect_attrs 表

连接属性是应用程序可以在连接时传递给服务器的键值对。对于基于`libmysqlclient`客户端库实现的 C API 的应用程序，`mysql_options()`和`mysql_options4()`函数定义了连接属性集。其他 MySQL 连接器可能提供其自己的属性定义方法。

这些性能模式表公开属性信息：

+   `session_account_connect_attrs`：当前会话及与会话帐户关联的其他会话的连接属性

+   `session_connect_attrs`：所有会话的连接属性

此外，写入审计日志的连接事件可能包括连接属性。请参阅第 8.4.5.4 节，“审计日志文件格式”。

以下划线（`_`）开头的属性名称保留供内部使用，不应由应用程序创建。这种约定允许 MySQL 引入新属性而不会与应用程序属性冲突，并使应用程序能够定义自己的属性，而不会与内部属性冲突。

+   可用连接属性

+   连接属性限制

#### 可用连接属性

在给定连接中可见的连接属性集取决于诸如您的平台、用于建立连接的 MySQL 连接器或客户端程序等因素。

`libmysqlclient`客户端库设置这些属性：

+   `_client_name`: 客户端名称（客户端库为`libmysql`）。

+   `_client_version`: 客户端库版本。

+   `_os`: 操作系统（例如，`Linux`，`Win64`）。

+   `_pid`: 客户端进程 ID。

+   `_platform`: 机器平台（例如，`x86_64`）。

+   `_thread`: 客户端线程 ID（仅限 Windows）。

其他 MySQL 连接器可能定义自己的连接属性。

MySQL Connector/C++ 8.0.16 及更高版本为使用 X DevAPI 或 X DevAPI for C 的应用程序定义了这些属性：

+   `_client_license`: 连接器许可证（例如 `GPL-2.0`）。

+   `_client_name`: 连接器名称（`mysql-connector-cpp`）。

+   `_client_version`: 连接器版本。

+   `_os`: 操作系统（例如，`Linux`，`Win64`）。

+   `_pid`: 客户端进程 ID。

+   `_platform`: 机器平台（例如，`x86_64`）。

+   `_source_host`: 客户端运行的机器的主机名。

+   `_thread`: 客户端线程 ID（仅限 Windows）。

MySQL Connector/J 定义了这些属性：

+   `_client_name`: 客户端名称

+   `_client_version`: 客户端库版本

+   `_os`: 操作系统（例如，`Linux`，`Win64`）

+   `_client_license`: 连接器许可证类型

+   `_platform`: 机器平台（例如，`x86_64`）

+   `_runtime_vendor`: Java 运行环境（JRE）供应商

+   `_runtime_version`: Java 运行环境（JRE）版本

MySQL Connector/NET 定义了这些属性：

+   `_client_version`: 客户端库版本。

+   `_os`: 操作系统（例如，`Linux`，`Win64`）。

+   `_pid`: 客户端进程 ID。

+   `_platform`: 机器平台（例如，`x86_64`）。

+   `_program_name`: 客户端名称。

+   `_thread`: 客户端线程 ID（仅限 Windows）。

Connector/Python 8.0.17 及更高版本的实现定义了这些属性；某些值和属性取决于 Connector/Python 的实现（纯 Python 或 c-ext）：

+   `_client_license`: 连接器的许可证类型；`GPL-2.0` 或 `Commercial`。（仅限纯 Python）

+   `_client_name`: 设置为 `mysql-connector-python`（纯 Python）或 `libmysql`（c-ext）

+   `_client_version`: 连接器版本（纯 Python）或 mysqlclient 库版本（c-ext）。

+   `_os`: 带有连接器的操作系统（例如，`Linux`，`Win64`）。

+   `_pid`: 源机器上的进程标识符（例如，`26955`）

+   `_platform`: 机器平台（例如，`x86_64`）。

+   `_source_host`: 连接器连接的机器的主机名。

+   `_connector_version`: 连接器版本（例如，`8.0.36`）（仅限 c-ext）。

+   `_connector_license`: 连接器的许可证类型；`GPL-2.0` 或 `Commercial`（仅限 c-ext）。

+   `_connector_name`: 始终设置为 `mysql-connector-python`（仅限 c-ext）。

PHP 定义了取决于编译方式的属性：

+   使用 `libmysqlclient` 编译：标准的 `libmysqlclient` 属性，前面已描述。

+   使用 `mysqlnd` 编译：仅 `_client_name` 属性，值为 `mysqlnd`。

许多 MySQL 客户端程序将 `program_name` 属性设置为与客户端名称相等的值。例如，**mysqladmin** 和 **mysqldump** 分别将 `program_name` 设置为 `mysqladmin` 和 `mysqldump`。MySQL Shell 将 `program_name` 设置为 `mysqlsh`。

一些 MySQL 客户端程序定义了额外的属性：

+   **mysql**（截至 MySQL 8.0.17）：

    +   `os_user`: 运行该程序的操作系统用户的名称。在 Unix 和类 Unix 系统以及 Windows 上可用。

    +   `os_sudouser`: `SUDO_USER`环境变量的值。在 Unix 和类 Unix 系统上可用。

    **mysql**连接属性的值为空时不会发送。

+   **mysqlbinlog**:

    +   `_client_role`: `binary_log_listener`

+   复制连接：

    +   `program_name`: `mysqld`

    +   `_client_role`: `binary_log_listener`

    +   `_client_replication_channel_name`: 通道名称。

+   `FEDERATED`存储引擎连接：

    +   `program_name`: `mysqld`

    +   `_client_role`: `federated_storage`

#### 连接属性限制

从客户端到服务器传输的连接属性数据存在限制：

+   在连接之前由客户端施加的固定限制。

+   在连接时由服务器施加的固定限制。

+   在连接时由性能模式施加的可配置限制。

对使用 C API 发起的连接，`libmysqlclient`库在客户端端对连接属性数据的总大小施加了 64KB 的限制：导致超出此限制的`mysql_options()`调用会产生`CR_INVALID_PARAMETER_NO`错误。其他 MySQL 连接器可能对客户端传输到服务器的连接属性数据量施加自己的限制。

在服务器端，对连接属性数据的大小进行以下检查：

+   服务器对其接受的连接的连接属性数据总大小施加 64KB 的限制。如果客户端尝试发送超过 64KB 的属性数据，服务器会拒绝连接。否则，服务器会认为属性缓冲区有效，并跟踪最长缓冲区的大小在`Performance_schema_session_connect_attrs_longest_seen`状态变量中。

+   对于接受的连接，性能模式会检查连接属性大小总和是否超过`performance_schema_session_connect_attrs_size`系统变量的值。如果属性大小超过此值，将会执行以下操作：

    +   性能模式截断属性数据并增加`Performance_schema_session_connect_attrs_lost`状态变量，该变量表示发生属性截断的连接数。

    +   如果`log_error_verbosity`系统变量大于 1，性能模式会向错误日志写入一条消息：

        ```sql
        Connection attributes of length *N* were truncated
        (*N* bytes lost)
        for connection *N*, user *user_name*@*host_name*
        (as *user_name*), auth: {yes|no}
        ```

        警告消息中的信息旨在帮助数据库管理员识别发生属性截断的客户端。

    +   一个`_truncated`属性被添加到会话属性中，其值表示丢失了多少字节，如果属性缓冲区有足够的空间。这使得性能模式能够在连接属性表中公开每个连接的截断信息。这些信息可以在不必检查错误日志的情况下进行检查。
