> 原文：[`dev.mysql.com/doc/refman/8.0/en/audit-log-reference.html`](https://dev.mysql.com/doc/refman/8.0/en/audit-log-reference.html)

#### 8.4.5.11 审计日志参考

以下各节提供了 MySQL 企业审计元素的参考：

+   审计日志表

+   审计日志功能

+   审计日志选项和变量参考

+   审计日志选项和变量

+   审计日志状态变量

要安装审计日志表和功能，请使用第 8.4.5.2 节，“安装或卸载 MySQL 企业审计”中提供的说明。除非安装了这些对象，否则`audit_log`插件将以传统模式运行（在 MySQL 8.0.34 中已弃用）。请参阅第 8.4.5.10 节，“传统模式审计日志过滤”。

##### 审计日志表

MySQL 企业审计使用`mysql`系统数据库中的表来持久存储过滤器和用户帐户数据。只有具有该数据库权限的用户才能访问这些表。要使用不同的数据库，请在服务器启动时设置`audit_log_database`系统变量。这些表使用`InnoDB`存储引擎。

如果这些表缺失，`audit_log`插件将以（已弃用的）传统模式运行。请参阅第 8.4.5.10 节，“传统模式审计日志过滤”。

`audit_log_filter`表存储过滤器定义。该表具有以下列：

+   `名称`

    过滤器名称。

+   `FILTER`

    与过滤器名称相关联的过滤器定义。定义存储为`JSON`值。

`audit_log_user`表存储用户帐户信息。该表具有以下列：

+   `用户`

    帐户的用户名部分。对于帐户`user1@localhost`，`USER`部分是`user1`。

+   `HOST`

    帐户的主机名部分。对于帐户`user1@localhost`，`HOST`部分是`localhost`。

+   `FILTERNAME`

    分配给帐户的过滤器名称。过滤器名称将帐户与`audit_log_filter`表中定义的过滤器关联起来。

##### 审计日志功能

本节描述了每个审计日志功能的目的、调用顺序和返回值。有关这些功能可以在何种条件下调用的信息，请参阅第 8.4.5.7 节，“审计日志过滤”。

每个审计日志函数都会返回一个字符串，指示操作是否成功。`OK`表示成功。`ERROR: *`message`*`表示失败。

截至 MySQL 8.0.19，审计日志函数将字符串参数转换为`utf8mb4`，并且字符串返回值为`utf8mb4`字符串。在 MySQL 8.0.19 之前，审计日志函数将字符串参数视为二进制字符串（这意味着它们不区分大小写），并且字符串返回值为二进制字符串。

如果从**mysql**客户端调用审计日志函数，则二进制字符串结果将使用十六进制表示，具体取决于`--binary-as-hex`的值。有关该选项的更多信息，请参见第 6.5.1 节，“mysql — MySQL 命令行客户端”。

可用的审计日志函数包括：

+   [`audit_log_encryption_password_get([*`keyring_id`*])`](audit-log-reference.html#function_audit-log-encryption-password-get)

    此函数从 MySQL 钥匙环中获取审计日志加密密码，必须启用，否则将出现错误。可以使用任何钥匙环组件或插件；有关说明，请参见第 8.4.4 节，“MySQL 钥匙环”。

    没有参数时，函数将检索当前加密密码作为二进制字符串。可以给定参数以指定要检索的哪个审计日志加密密码。参数必须是当前密码或存档密码的钥匙环 ID。

    有关审计日志加密的更多信息，请参见加密审计日志文件。

    参数：

    *`keyring_id`*：从 MySQL 8.0.17 开始，这个可选参数表示要检索的密码的钥匙环 ID。最大允许长度为 766 字节。如果省略，函数将检索当前密码。

    在 MySQL 8.0.17 之前，不允许有任何参数。该函数始终检索当前密码。

    返回值：

    成功的密码字符串（最多 766 字节），或失败时为`NULL`和错误。

    示例：

    检索当前密码：

    ```sql
    mysql> SELECT audit_log_encryption_password_get();
    +-------------------------------------+
    | audit_log_encryption_password_get() |
    +-------------------------------------+
    | secret                              |
    +-------------------------------------+
    ```

    要通过 ID 检索密码，可以通过查询性能模式`keyring_keys`表来确定存在哪些审计日志钥匙环 ID：

    ```sql
    mysql> SELECT KEY_ID FROM performance_schema.keyring_keys
           WHERE KEY_ID LIKE 'audit_log%'
           ORDER BY KEY_ID;
    +-----------------------------+
    | KEY_ID                      |
    +-----------------------------+
    | audit_log-20190415T152248-1 |
    | audit_log-20190415T153507-1 |
    | audit_log-20190416T125122-1 |
    | audit_log-20190416T141608-1 |
    +-----------------------------+
    mysql> SELECT audit_log_encryption_password_get('audit_log-20190416T125122-1');
    +------------------------------------------------------------------+
    | audit_log_encryption_password_get('audit_log-20190416T125122-1') |
    +------------------------------------------------------------------+
    | segreto                                                          |
    +------------------------------------------------------------------+
    ```

+   `audit_log_encryption_password_set(*`password`*)`

    将当前审计日志加密密码设置为参数，并将密码存储在 MySQL 钥匙环中。截至 MySQL 8.0.19，密码存储为`utf8mb4`字符串。在 MySQL 8.0.19 之前，密码以二进制形式存储。

    如果启用了加密，此函数将执行一个日志文件旋转操作，重命名当前日志文件，并开始一个新的日志文件，使用密码加密。必须启用密钥环，否则将出现错误。可以使用任何密钥环组件或插件；有关说明，请参见第 8.4.4 节，“MySQL 密钥环”。

    有关审计日志加密的更多信息，请参见加密审计日志文件。

    参数：

    *`password`*: 密码字符串。最大允许长度为 766 字节。

    返回值：

    成功返回 1，失败返回 0。

    示例：

    ```sql
    mysql> SELECT audit_log_encryption_password_set(*password*);
    +---------------------------------------------+
    | audit_log_encryption_password_set(*password*) |
    +---------------------------------------------+
    | 1                                           |
    +---------------------------------------------+
    ```

+   `audit_log_filter_flush()`

    调用任何其他过滤函数会立即影响操作审计日志过滤，并更新审计日志表。如果您改为直接修改这些表的内容，使用诸如`INSERT`、`UPDATE`和`DELETE`等语句，更改不会立即影响过滤。要刷新更改并使其生效，请调用`audit_log_filter_flush()`。

    警告

    只有在直接修改审计表后，才应使用`audit_log_filter_flush()`来强制重新加载所有过滤器。否则，应避免使用此函数。实际上，这是使用`UNINSTALL PLUGIN`加`INSTALL PLUGIN`卸载和重新加载`audit_log`插件的简化版本。

    `audit_log_filter_flush()`会影响所有当前会话，并将它们从先前的过滤器中分离。当前会话不再被记录，除非它们断开连接并重新连接，或执行更改用户操作。

    如果此函数失败，将返回错误消息，并且审计日志将被禁用，直到下一次成功调用`audit_log_filter_flush()`。

    参数：

    无。

    返回值：

    一个指示操作是否成功的字符串。`OK`表示成功。`ERROR: *`message`*`表示失败。

    示例：

    ```sql
    mysql> SELECT audit_log_filter_flush();
    +--------------------------+
    | audit_log_filter_flush() |
    +--------------------------+
    | OK                       |
    +--------------------------+
    ```

+   `audit_log_filter_remove_filter(*`filter_name`*)`

    给定一个过滤器名称，从当前过滤器集中移除该过滤器。过滤器不存在也不会报错。

    如果已删除的过滤器分配给任何用户帐户，则这些用户将停止被过滤（它们将从`audit_log_user`表中删除）。停止过滤包括这些用户的任何当前会话：它们将从过滤器中分离，并且不再被记录。

    参数：

    +   *`filter_name`*：指定过滤器名称的字符串。

    返回值：

    一个指示操作是否成功的字符串。`OK`表示成功。`ERROR: *`message`*`表示失败。

    示例：

    ```sql
    mysql> SELECT audit_log_filter_remove_filter('SomeFilter');
    +----------------------------------------------+
    | audit_log_filter_remove_filter('SomeFilter') |
    +----------------------------------------------+
    | OK                                           |
    +----------------------------------------------+
    ```

+   `audit_log_filter_remove_user(*`user_name`*)`

    给定用户账户名称，使用户不再分配到过滤器。如果用户没有分配过滤器，则不会出错。用户的当前会话过滤保持不受影响。如果存在默认账户过滤器，则使用默认账户过滤器过滤用户的新连接，否则不记录。

    如果名称为`%`，则函数会移除用于没有明确分配过滤器的任何用户账户的默认账户过滤器。

    参数：

    +   *`user_name`*：用户账户名称，以`*`user_name`*@*`host_name`*`格式的字符串表示，或者使用`%`表示默认账户。

    返回值：

    一个指示操作是否成功的字符串。`OK`表示成功。`ERROR: *`message`*`表示失败。

    示例：

    ```sql
    mysql> SELECT audit_log_filter_remove_user('user1@localhost');
    +-------------------------------------------------+
    | audit_log_filter_remove_user('user1@localhost') |
    +-------------------------------------------------+
    | OK                                              |
    +-------------------------------------------------+
    ```

+   `audit_log_filter_set_filter(*`filter_name`*, *`definition`*)`

    给定过滤器名称和定义，将过滤器添加到当前过滤器集中。如果过滤器已经存在并且被当前会话使用，那么这些会话将与过滤器分离，并且不再记录。这是因为新的过滤器定义具有与先前 ID 不同的新过滤器 ID。

    参数：

    +   *`filter_name`*：指定过滤器名称的字符串。

    +   *`definition`*：指定过滤器定义的`JSON`值。

    返回值：

    一个指示操作是否成功的字符串。`OK`表示成功。`ERROR: *`message`*`表示失败。

    示例：

    ```sql
    mysql> SET @f = '{ "filter": { "log": false } }';
    mysql> SELECT audit_log_filter_set_filter('SomeFilter', @f);
    +-----------------------------------------------+
    | audit_log_filter_set_filter('SomeFilter', @f) |
    +-----------------------------------------------+
    | OK                                            |
    +-----------------------------------------------+
    ```

+   `audit_log_filter_set_user(*`user_name`*, *`filter_name`*)`

    给定用户账户名称和过滤器名称，将过滤器分配给用户。用户只能分配一个过滤器，因此如果用户已经分配了过滤器，则分配将被替换。用户的当前会话过滤保持不受影响。新连接将使用新的过滤器进行过滤。

    作为特例，名称`%`代表默认账户。该过滤器用于来自没有明确分配过滤器的任何用户账户的连接。

    参数：

    +   *`user_name`*：用户账户名称，以`*`user_name`*@*`host_name`*`格式的字符串表示，或者使用`%`表示默认账户。

    +   *`filter_name`*：指定过滤器名称的字符串。

    返回值：

    一个指示操作是否成功的字符串。`OK`表示成功。`ERROR: *`message`*`表示失败。

    示例：

    ```sql
    mysql> SELECT audit_log_filter_set_user('user1@localhost', 'SomeFilter');
    +------------------------------------------------------------+
    | audit_log_filter_set_user('user1@localhost', 'SomeFilter') |
    +------------------------------------------------------------+
    | OK                                                         |
    +------------------------------------------------------------+
    ```

+   [`audit_log_read([*`arg`*])`](audit-log-reference.html#function_audit-log-read)

    读取审计日志并返回一个`JSON` 字符串结果。如果审计日志格式不是`JSON`，则会出错。

    没有参数或者一个`JSON` 哈希参数时，`audit_log_read()` 从审计日志中读取事件，并返回一个包含审计事件数组的`JSON` 字符串。哈希参数中的项会影响读取过程，稍后会描述。返回数组中的每个元素都是一个以`JSON` 哈希表示的事件，除了最后一个元素可能是一个`JSON` `null` 值，表示没有更多事件可供读取。

    使用由`JSON` `null` 值组成的参数时，`audit_log_read()` 将关闭当前读取序列。

    有关审计日志读取过程的更多详细信息，请参见第 8.4.5.6 节，“读取审计日志文件”。

    参数：

    要获取最近写入事件的书签，请调用`audit_log_read_bookmark()`。

    *`arg`*: 参数是可选的。如果省略，函数将从当前位置读取事件。如果存在，参数可以是一个`JSON` `null` 值以关闭读取序列，或者一个`JSON` 哈希。在哈希参数中，项是可选的，并控制读取操作的方面，例如从哪个位置开始读取或者要读取多少事件。以下项是重要的（其他项将被忽略）：

    +   `start`: 审计日志中要读取的第一个事件的位置。位置以时间戳给出，并且从时间戳值之后发生的第一个事件开始读取。`start` 项的格式如下，其中*`value`*是一个字面时间戳值：

        ```sql
        "start": { "timestamp": "*value*" }
        ```

        `start` 项自 MySQL 8.0.22 版本开始允许使用。

    +   `timestamp`，`id`: 要读取的第一个事件在审计日志中的位置。`timestamp` 和 `id` 项一起构成一个书签，唯一标识特定事件。如果`audit_log_read()` 参数包含任一项，则必须同时包含两项才能完全指定位置，否则会出错。

    +   `max_array_length`: 从日志中读取的最大事件数。如果省略此项，则默认读取到日志末尾或者直到读取缓冲区已满为止。

    要指定给`audit_log_read()`的起始位置，传递一个包含`start`项或由`时间戳`和`id`项组成的书签的哈希参数。如果哈希参数同时包含`start`项和书签，则会出现错误。

    如果哈希参数未指定起始位置，则从当前位置继续读取。

    如果时间戳值不包含时间部分，则假定时间部分为`00:00:00`。

    返回值：

    如果调用成功，返回值是包含审核事件数组的`JSON`字符串，或者如果将其作为参数传递以关闭读取序列，则返回`JSON` `null`值。如果调用失败，返回值为`NULL`，并出现错误。

    示例：

    ```sql
    mysql> SELECT audit_log_read(audit_log_read_bookmark());
    +-----------------------------------------------------------------------+
    | audit_log_read(audit_log_read_bookmark())                             |
    +-----------------------------------------------------------------------+
    |  {"timestamp":"2020-05-18 22:41:24","id":0,"class":"connection", ... |
    +-----------------------------------------------------------------------+
    mysql> SELECT audit_log_read('null');
    +------------------------+
    | audit_log_read('null') |
    +------------------------+
    | null                   |
    +------------------------+
    ```

    注意：

    在 MySQL 8.0.19 之前，字符串返回值是二进制的[`JSON`字符串。有关将这些值转换为非二进制字符串的信息，请参见第 8.4.5.6 节，“读取审计日志文件”。

+   `audit_log_read_bookmark()`

    返回一个表示最近写入的审计日志事件的`JSON`字符串书签。如果审计日志格式不是`JSON`，则会出现错误。

    书签是一个包含`时间戳`和`id`项的`JSON`哈希，用于唯一标识审核日志中事件的位置。适合传递给`audit_log_read()`以指示该函数开始读取的位置。

    有关审计日志读取过程的更多详细信息，请参见第 8.4.5.6 节，“读取审计日志文件”。

    参数：

    无。

    返回值：

    包含成功书签的`JSON`字符串，或包含失败的`NULL`和错误。

    示例：

    ```sql
    mysql> SELECT audit_log_read_bookmark();
    +-------------------------------------------------+
    | audit_log_read_bookmark()                       |
    +-------------------------------------------------+
    | { "timestamp": "2019-10-03 21:03:44", "id": 0 } |
    +-------------------------------------------------+
    ```

    注意：

    在 MySQL 8.0.19 之前，字符串返回值是二进制的`JSON`字符串。有关将这些值转换为非二进制字符串的信息，请参见第 8.4.5.6 节，“读取审计日志文件”。

+   `audit_log_rotate()`

    参数：

    无。

    返回值：

    重命名后的文件名。

    示例：

    ```sql
    mysql> SELECT audit_log_rotate();
    ```

    使用`audit_log_rotate()`需要`AUDIT_ADMIN`权��。

##### 审计日志选项和变量参考

**表 8.44 审计日志选项和变量参考**

| 名称 | 命令行 | 选项文件 | 系统变量 | 状态变量 | 变量范围 | 动态 |
| --- | --- | --- | --- | --- | --- | --- |
| audit-log | 是 | 是 |  |  |  |  |
| 审计日志缓冲区大小 | 是 | 是 | 是 |  | 全局 | 否 |
| 审计日志压缩 | 是 | 是 | 是 |  | 全局 | 否 |
| 审计日志连接策略 | 是 | 是 | 是 |  | 全局 | 是 |
| 当前审计日志会话 |  |  | 是 |  | 两者 | 否 |
| 审计日志当前大小 |  |  |  | 是 | 全局 | 否 |
| 审计日志数据库 | 是 | 是 | 是 |  | 全局 | 否 |
| 禁用审计日志 | 是 | 是 | 是 |  | 全局 | 是 |
| 审计日志加密 | 是 | 是 | 是 |  | 全局 | 否 |
| 审计日志事件最大丢弃大小 |  |  |  | 是 | 全局 | 否 |
| 审计日志事件 |  |  |  | 是 | 全局 | 否 |
| 已过滤审计日志事件 |  |  |  | 是 | 全局 | 否 |
| 已丢失审计日志事件 |  |  |  | 是 | 全局 | 否 |
| 已写入审计日志事件 |  |  |  | 是 | 全局 | 否 |
| 排除账户的审计日志 | 是 | 是 | 是 |  | 全局 | 是 |
| 审计日志文件 | 是 | 是 | 是 |  | 全局 | 否 |
| 审计日志过滤器 ID |  |  | 是 |  | 两者 | 否 |
| 审计日志刷新 |  |  | 是 |  | 全局 | 是 |
| 审计日志刷新间隔秒数 | 是 |  | 是 |  | 全局 | 否 |
| 审计日志格式 | 是 | 是 | 是 |  | 全局 | 否 |
| 包含账户的审计日志 | 是 | 是 | 是 |  | 全局 | 是 |
| 审计日志最大大小 | 是 | 是 | 是 |  | 全局 | 是 |
| 审计日志密码历史保留天数 | 是 | 是 | 是 |  | 全局 | 是 |
| 审计日志策略 | 是 | 是 | 是 |  | 全局 | �� |
| 审计日志修剪秒数 | 是 | 是 | 是 |  | 全局 | 是 |
| 审计日志读取缓冲区大小 | 是 | 是 | 是 |  | 变化 | 变化 |
| audit_log_rotate_on_size | 是 | 是 | 是 |  | 全局 | 是 |
| audit_log_statement_policy | 是 | 是 | 是 |  | 全局 | 是 |
| audit_log_strategy | 是 | 是 | 是 |  | 全局 | 否 |
| Audit_log_total_size |  |  |  | 是 | 全局 | 否 |
| Audit_log_write_waits |  |  |  | 是 | 全局 | 否 |
| 名称 | 命令行 | 选项文件 | 系统变量 | 状态变量 | 变量范围 | 动态 |

##### 审计日志选项和变量

本节描述了配置 MySQL Enterprise Audit 操作的命令选项和系统变量。如果在启动时指定的值不正确，`audit_log` 插件可能无法正确初始化，服务器也不会加载它。在这种情况下，服务器可能还会因为无法识别其他审计日志设置而产生错误消息。

要配置审计日志插件的激活，请使用此选项：

+   [`--audit-log[=*`value`*]`](audit-log-reference.html#option_mysqld_audit-log)

    | 命令行格式 | `--audit-log[=value]` |
    | --- | --- |
    | 类型 | 枚举 |
    | 默认值 | `ON` |
    | 有效值 | `ON``OFF``FORCE``FORCE_PLUS_PERMANENT` |

    此选项控制服务器在启动时如何加载 `audit_log` 插件。只有在插件之前已经使用 `INSTALL PLUGIN` 注册过或者使用 `--plugin-load` 或 `--plugin-load-add` 加载过插件时才可用。参见 Section 8.4.5.2, “Installing or Uninstalling MySQL Enterprise Audit”。

    选项值应该是插件加载选项中可用的值之一，如 Section 7.6.1, “Installing and Uninstalling Plugins” 中所述。例如，`--audit-log=FORCE_PLUS_PERMANENT` 告诉服务器加载插件并防止在服务器运行时被移除。

如果启用了审计日志插件，它会暴露几个系统变量，允许对日志进行控制：

```sql
mysql> SHOW VARIABLES LIKE 'audit_log%';
+--------------------------------------+--------------+
| Variable_name                        | Value        |
+--------------------------------------+--------------+
| audit_log_buffer_size                | 1048576      |
| audit_log_compression                | NONE         |
| audit_log_connection_policy          | ALL          |
| audit_log_current_session            | OFF          |
| audit_log_database                   | mysql        |
| audit_log_disable                    | OFF          |
| audit_log_encryption                 | NONE         |
| audit_log_exclude_accounts           |              |
| audit_log_file                       | audit.log    |
| audit_log_filter_id                  | 0            |
| audit_log_flush                      | OFF          |
| audit_log_flush_interval_seconds     | 0            |
| audit_log_format                     | NEW          |
| audit_log_format_unix_timestamp      | OFF          |
| audit_log_include_accounts           |              |
| audit_log_max_size                   | 0            |
| audit_log_password_history_keep_days | 0            |
| audit_log_policy                     | ALL          |
| audit_log_prune_seconds              | 0            |
| audit_log_read_buffer_size           | 32768        |
| audit_log_rotate_on_size             | 0            |
| audit_log_statement_policy           | ALL          |
| audit_log_strategy                   | ASYNCHRONOUS |
+--------------------------------------+--------------+
```

你可以在服务器启动时设置任何这些变量，有些变量可以在运行时设置。只能用于传统模式审计日志过滤的变量会有相应标注。

+   `audit_log_buffer_size`

    | 命令行格式 | `--audit-log-buffer-size=#` |
    | --- | --- |
    | 系统变量 | `audit_log_buffer_size` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1048576` |
    | 最小值 | `4096` |
    | 最大值（64 位平台） | `18446744073709547520` |
    | 最大值（32 位平台） | `4294967295` |
    | 单位 | 字节 |
    | 块大小 | `4096` |

    当审计日志插件异步写入事件到日志时，它使用一个缓冲区来存储事件内容以便写入。此变量控制该缓冲区的大小，以字节为单位。服务器将该值调整为 4096 的倍数。插件使用一个缓冲区，在初始化时分配，终止时删除。插件仅在日志记录是异步的情况下分配此缓冲区。

+   `audit_log_compression`

    | 命令行格式 | `--audit-log-compression=value` |
    | --- | --- |
    | 系统变量 | `audit_log_compression` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `NONE` |
    | 有效值 | `NONE``GZIP` |

    审计日志文件的压缩类型。允许的值为`NONE`（无压缩；默认值）���`GZIP`（GNU Zip 压缩）。有关更多信息，请参见压缩审计日志文件。 

+   `audit_log_connection_policy`

    | 命令行格式 | `--audit-log-connection-policy=value` |
    | --- | --- |
    | 已弃用 | 8.0.34 |
    | 系统变量 | `audit_log_connection_policy` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `ALL` |
    | 有效值 | `ALL``ERRORS``NONE` |

    注意

    此已弃用的变量仅适用于传统模式审计日志过滤（参见第 8.4.5.10 节，“传统模式审计日志过滤”）。

    控制审计日志插件将连接事件写入其日志文件的策略。以下表格显示了允许的值。

    | 值 | 描述 |
    | --- | --- |
    | `ALL` | 记录所有连接事件 |
    | `ERRORS` | 仅记录连接失败事件 |
    | `NONE` | 不记录连接事件 |

    注意

    在服务器启动时，如果还指定了`audit_log_policy`，则可能会覆盖对`audit_log_connection_policy`的任何显式值，如第 8.4.5.5 节，“配置审计日志特性”中所述。

+   `audit_log_current_session`

    | 系统变量 | `audit_log_current_session` |
    | --- | --- |
    | 作用范围 | 全局，会话 |
    | 动态 | 否 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `取决于过滤策略` |

    当前会话是否启用审计日志记录。此变量的会话值只读。它在会话开始时根据`audit_log_include_accounts`和`audit_log_exclude_accounts`系统变量的值设置。审计日志插件使用会话值来确定是否为会话审计事件。（有一个全局值，但插件不使用它。）

+   `audit_log_database`

    | 命令行格式 | `--audit-log-database=value` |
    | --- | --- |
    | 引入版本 | 8.0.33 |
    | 系统变量 | `audit_log_database` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 字符串 |
    | 默认值 | `mysql` |

    指定`audit_log`插件用于查找其表的数据库。此变量只读。更多信息，请参见第 8.4.5.2 节，“安装或卸载 MySQL Enterprise Audit”。

+   `audit_log_disable`

    | 命令行格式 | `--audit-log-disable[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.28 |
    | 系统变量 | `audit_log_disable` |
    | 作用范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    允许禁用所有连接和已连接会话的审计日志记录。除了`SYSTEM_VARIABLES_ADMIN`权限外，禁用审计日志记录还需要`AUDIT_ADMIN`权限。请参见第 8.4.5.9 节，“禁用审计日志记录”。

+   `audit_log_encryption`

    | 命令行格式 | `--audit-log-encryption=value` |
    | --- | --- |
    | 系统变量 | `audit_log_encryption` |
    | 作用范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` Hint Applies | 否 |
    | 类型 | 枚举 |
    | 默认值 | `NONE` |
    | 有效值 | `NONE``AES` |

    审计日志文件的加密类型。允许的值为`NONE`（无加密；默认值）和`AES`（AES-256-CBC 密码加密）。有关更多信息，请参见加密审计日志文件。

+   `audit_log_exclude_accounts`

    | 命令行格式 | `--audit-log-exclude-accounts=value` |
    | --- | --- |
    | 已弃用 | 8.0.34 |
    | 系统变量 | `audit_log_exclude_accounts` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `NULL` |

    注意

    此已弃用变量仅适用于传统模式审计日志过滤（请参见第 8.4.5.10 节，“传统模式审计日志过滤”）。

    不应记录事件的帐户。该值应为`NULL`或包含一个或多个逗号分隔帐户名称列表的字符串。有关更多信息，请参见第 8.4.5.7 节，“审计日志过滤”。

    对`audit_log_exclude_accounts`的修改仅影响在修改后创建的连接，而不影响现有连接。

+   `audit_log_file`

    | 命令行格式 | `--audit-log-file=file_name` |
    | --- | --- |
    | 系统变量 | `audit_log_file` |
    | 范围 | 全�� |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 文件名 |
    | 默认值 | `audit.log` |

    审计日志插件写入事件的文件的基本名称和后缀。默认值为`audit.log`，无论日志格式如何。要使名称后缀与格式对应，必须显式设置名称，选择不同的后缀（例如，对于 XML 格式为`audit.xml`，对于 JSON 格式为`audit.json`）。

    如果`audit_log_file`的值是相对路径名，则插件将其解释为相对于数据目录的路径。如果该值是完整路径名，则插件将直接使用该值。如果希望将审计文件定位到单独的文件系统或目录上，完整路径名可能会很有用。出于安全原因，将审计日志文件写入仅对 MySQL 服务器和有合理查看日志原因的用户可访问的目录。

    有关审计日志插件如何解释`audit_log_file`值以及插件初始化和终止时发生的文件重命名规则的详细信息，请参见审计日志文件命名约定。

    审计日志插件使用包含审计日志文件的目录（从`audit_log_file`的值确定）作为可读取审计日志文件的位置。从这些日志文件和当前文件中，插件构建了一个列表，其中包含适用于审计日志标记和读取功能的文件。参见第 8.4.5.6 节，“读取审计日志文件”。

+   `audit_log_filter_id`

    | 系统变量 | `audit_log_filter_id` |
    | --- | --- |
    | 作用域 | 全局，会话 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `1` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |

    此变量的会话值表示当前会话的内部维护的审计过滤器的 ID。值为 0 表示会话未分配任何过滤器。

+   `audit_log_flush`

    | 系统变量 | `audit_log_flush` |
    | --- | --- |
    | 作用域 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    注意

    `audit_log_flush`变量在 MySQL 8.0.31 中已被弃用；预计在未来的 MySQL 版本中将不再支持它。它已被`audit_log_rotate()`函数取代。

    如果`audit_log_rotate_on_size`为 0，则自动审计日志文件轮换被禁用，只有在手动执行时才会发生轮换。在这种情况下，通过将`audit_log_flush`设置为 1 或`ON`来启用它会导致审计日志插件关闭并重新打开其日志文件以刷新它。（变量值保持为`OFF`，因此您无需在再次启用它以执行另一个刷新之前显式禁用它。）有关更多信息，请参见第 8.4.5.5 节，“配置审计日志特性”。

+   `audit_log_flush_interval_seconds`

    | 命令行格式 | `--audit-log-flush-interval-seconds[=value]` |
    | --- | --- |
    | 引入版本 | 8.0.34 |
    | 系统变量 | `audit_log_flush_interval_seconds` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 无符号长整型 |
    | 默认值 | `0` |
    | 最大值（Windows） | `4294967295` |
    | 最大值（其他） | `18446744073709551615` |
    | 单位 | 秒 |

    此系统变量取决于必须安装和启用的`scheduler`组件（请参阅第 7.5.5 节，“调度器组件”）。要检查组件的状态：

    ```sql
    SHOW VARIABLES LIKE 'component_scheduler%';
    +-----------------------------+-------+
    | Variable_name               | Value |
    +-----------------------------+-------|
    | component_scheduler.enabled | On    |
    +-----------------------------+-------+
    ```

    当`audit_log_flush_interval_seconds`的值为零（默认值）时，即使启用了`scheduler`组件（`ON`），也不会自动刷新权限。

    不允许值为`1`和`59`；相反，这些值会自动调整为`60`，并且服务器会发出警告。大于`60`的值定义了`scheduler`组件从启动或上一次执行开始等待多少秒，直到尝试安排另一个执行。

    要将此全局系统变量持久化到`mysqld-auto.cnf`文件中而不设置全局变量运行时值，请在变量名称之前加上`PERSIST_ONLY`关键字或`@@PERSIST_ONLY.`限定符。

+   `audit_log_format`

    | 命令行格式 | `--audit-log-format=value` |
    | --- | --- |
    | 系统变量 | `audit_log_format` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `NEW` |
    | 有效值 | `OLD``NEW``JSON` |

    审计日志文件格式。允许的值为`OLD`（旧式 XML）、`NEW`（新式 XML；默认值）和`JSON`。有关每种格式的详细信息，请参阅第 8.4.5.4 节，“审计日志文件格式”。

+   `audit_log_format_unix_timestamp`

    | 命令行格式 | `--audit-log-format-unix-timestamp[={OFF&#124;ON}]` |
    | --- | --- |
    | 引入版本 | 8.0.26 |
    | 系统变量 | `audit_log_format_unix_timestamp` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    此变量仅适用于 JSON 格式的审计日志输出。当为真时，启用此变量会导致每个日志文件记录包含一个`time`字段。该字段值是一个整数，表示生成审计事件的日期和时间的 UNIX 时间戳值。

    在运行时更改此变量的值会导致日志文件轮换，以便对于给定的 JSON 格式日志文件，文件中的所有记录要么包含`time`字段，要么不包含。

    在运行时设置`audit_log_format_unix_timestamp`的值需要`AUDIT_ADMIN`权限，除了通常需要设置全局系统变量运行时值的`SYSTEM_VARIABLES_ADMIN`权限（或已弃用的`SUPER`权限）。

+   `audit_log_include_accounts`

    | 命令行格式 | `--audit-log-include-accounts=value` |
    | --- | --- |
    | 已弃用 | 8.0.34 |
    | 系统变量 | `audit_log_include_accounts` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 字符串 |
    | 默认值 | `NULL` |

    注意

    此已弃用变量仅适用于传统模式的审计日志过滤（参见第 8.4.5.10 节，“传统模式审计日志过滤”）。

    应记录事件的帐户。该值应为`NULL`或包含一个或多个逗号分隔帐户名称列表的字符串。有关更多信息，请参见第 8.4.5.7 节，“审计日志过滤”。

    对`audit_log_include_accounts`的修改仅影响在修改后创建的连接，而不影响现有连接。

+   `audit_log_max_size`

    | 命令行格式 | `--audit-log-max-size=#` |
    | --- | --- |
    | 引入 | 8.0.26 |
    | 系统变量 | `audit_log_max_size` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值（Windows） | `4294967295` |
    | 最大值（其他） | `18446744073709551615` |
    | 单位 | 字节 |
    | 块大小 | `4096` |

    `audit_log_max_size`涉及仅支持 JSON 格式日志文件的日志文件修剪。它控制基于组合日志文件大小的修剪：

    +   值为 0（默认）会禁用基于大小的修剪。不强制执行大小限制。

    +   值大于 0 会启用基于大小的修剪。该值是超过该值后日志文件变得需要修剪的组合大小。

    如果将`audit_log_max_size`设置为不是 4096 的倍数的值，则会被截断为最接近的倍数。特别是，将其设置为小于 4096 的值会将其设置为 0，不会发生基于大小的修剪。

    如果`audit_log_max_size`和`audit_log_rotate_on_size`都大于 0，则`audit_log_max_size`应该大于`audit_log_rotate_on_size`的值的 7 倍以上。否则，会向服务器错误日志写入警告，因为在这种情况下，基于大小的修剪的“粒度”可能不足以防止每次发生时删除所有或大部分旋转的日志文件。

    注意

    设置`audit_log_max_size`本身并不足以导致日志文件修剪，因为修剪算法同时使用`audit_log_rotate_on_size`，`audit_log_max_size`和`audit_log_prune_seconds`。详情请参见审计日志文件的空间管理。

+   `audit_log_password_history_keep_days`

    | 命令行格式 | `--audit-log-password-history-keep-days=#` |
    | --- | --- |
    | 引入版本 | 8.0.17 |
    | 系统变量 | `audit_log_password_history_keep_days` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `4294967295` |
    | 单位 | 天 |

    审计日志插件使用存储在 MySQL 密钥环中的加密密码实现日志文件加密（请参阅加密审计日志文件）。该插件还实现了密码历史记录，包括密码存档和过期（删除）。

    当审计日志插件创建新的加密密码时，它会存档先前的密码（如果存在），以供以后使用。`audit_log_password_history_keep_days`变量控制过期存档密码的自动删除。其值表示存档的审计日志加密密码在多少天后被删除。默认值为 0，禁用密码过期：密码保留期永远。

    在以下情况下创建新的审计日志加密密码：

    +   在插件初始化期间，如果插件发现启用了日志文件加密，则会检查密钥环是否包含审计日志加密密码。如果没有，则插件会自动生成一个随机初始加密密码。

    +   当调用`audit_log_encryption_password_set()`函数设置特定密码时。

    在每种情况下，插件将新密码存储在密钥环中，并用它来加密新的日志文件。

    在以下情况下移除过期的审计日志加密密码：

    +   在插件初始化期间。

    +   当调用`audit_log_encryption_password_set()`函数时。

    +   当`audit_log_password_history_keep_days`的运行时值从当前值更改为大于 0 的值时。运行时值更改发生在使用`GLOBAL`或`PERSIST`关键字的`SET`语句，但不适用于`PERSIST_ONLY`关键字。`PERSIST_ONLY`将变量设置写入`mysqld-auto.cnf`，但不影响运行时值。

    当密码移除发生时，当前值`audit_log_password_history_keep_days`决定要移除哪些密码：

    +   如果值为 0，则插件不会删除任何密码。

    +   如果值为*`N`* > 0，则插件会删除超过*`N`*天的密码。

    注意

    注意不要过期仍然需要读取已归档的加密日志文件的旧密码。

    如果通常禁用密码过期（即`audit_log_password_history_keep_days`的值为 0），可以通过临时分配大于零的值来执行按需清理操作。例如，要使超过 365 天的密码过期，请执行以下操作：

    ```sql
    SET GLOBAL audit_log_password_history_keep_days = 365;
    SET GLOBAL audit_log_password_history_keep_days = 0;
    ```

    设置运行时值`audit_log_password_history_keep_days`需要`AUDIT_ADMIN`权限，除了通常需要设置全局系统变量运行时值的`SYSTEM_VARIABLES_ADMIN`权限（或已弃用的`SUPER`权限）。

+   `audit_log_policy`

    | 命令行格式 | `--audit-log-policy=value` |
    | --- | --- |
    | 已弃用 | 8.0.34 |
    | 系统变量 | `audit_log_policy` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `ALL` |
    | 有效值 | `ALL``LOGINS``QUERIES``NONE` |

    注意

    此弃用变量仅适用于传统模式审计日志过滤（参见第 8.4.5.10 节，“传统模式审计日志过滤”）。

    控制审计日志插件将事件写入其日志文件的策略。以下表格显示了允许的值。

    | 值 | 描述 |
    | --- | --- |
    | `ALL` | 记录所有事件 |
    | `LOGINS` | 仅记录登录事件 |
    | `QUERIES` | 仅记录查询事件 |
    | `NONE` | 什么都不记录（禁用审计流） |

    `audit_log_policy`只能在服务器启动时设置。在运行时，它是一个只读变量。另外两个系统变量，`audit_log_connection_policy`和`audit_log_statement_policy`，提供对日志记录策略的更精细控制，可以在启动时或运行时设置。如果在启动时使用`audit_log_policy`而不是其他两个变量，则服务器会使用其值来设置这些变量。有关策略变量及其交互的更多信息，请参见第 8.4.5.5 节，“配置审计日志特性”。

+   `audit_log_prune_seconds`

    | 命令行格式 | `--audit-log-prune-seconds=#` |
    | --- | --- |
    | 引入版本 | 8.0.24 |
    | 系统变量 | `audit_log_prune_seconds` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值（Windows） | `4294967295` |
    | 最大值（其他） | `18446744073709551615` |
    | 单位 | 字节 |

    `audit_log_prune_seconds`涉及审计日志文件修剪，仅支持 JSON 格式的日志文件。它根据日志文件的年龄控制修剪：

    +   值为 0（默认值）会禁用基于年龄的修剪。不强制执行年龄限制。

    +   值大于 0 会启用基于年龄的修剪。该值是日志文件变得需要修剪的秒数。

    注意

    单独设置`audit_log_prune_seconds` 是不足以导致日志文件修剪发生的，因为修剪算法同时使用`audit_log_rotate_on_size`、`audit_log_max_size` 和`audit_log_prune_seconds`。有关详细信息，请参见 审计日志文件的空间管理。

+   `audit_log_read_buffer_size`

    | 命令行格式 | `--audit-log-read-buffer-size=#` |
    | --- | --- |
    | 系统变量 | `audit_log_read_buffer_size` |
    | 范围 (≥ 8.0.12) | 全局, 会话 |
    | 范围 (8.0.11) | 全局 |
    | 动态 (≥ 8.0.12) | 是 |
    | 动态 (8.0.11) | 否 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 (≥ 8.0.12) | `32768` |
    | 默认值 (8.0.11) | `1048576` |
    | 最小值 (≥ 8.0.12) | `32768` |
    | 最小值 (8.0.11) | `1024` |
    | 最大值 | `4194304` |
    | 单位 | 字节 |

    从审计日志文件中读取的缓冲区大小，以字节为单位。`audit_log_read()` 函数最多读取这么多字节。仅支持 JSON 日志格式的日志文件读取。有关更多信息，请参见 第 8.4.5.6 节，“读取审计日志文件”。

    截至 MySQL 8.0.12，此变量默认为 32KB，并可在运行时设置。每个客户端应适当设置其`audit_log_read_buffer_size`的会话值，以供其使用`audit_log_read()`。在 MySQL 8.0.12 之前，`audit_log_read_buffer_size` 默认为 1MB，影响所有客户端，并且只能在服务器启动时更改。

+   `audit_log_rotate_on_size`

    | 命令行格式 | `--audit-log-rotate-on-size=#` |
    | --- | --- |
    | 系统变量 | `audit_log_rotate_on_size` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR` 提示适用 | 否 |
    | 类型 | 整数 |
    | 默认值 | `0` |
    | 最小值 | `0` |
    | 最大值 | `18446744073709551615` |
    | 单位 | 字节 |
    | 块大小 | `4096` |

    如果`audit_log_rotate_on_size`为 0，则审计日志插件不执行基于大小的自动日志文件轮换。如果要进行旋转，必须手动执行；请参见手动审计日志文件旋转（MySQL 8.0.31 之前）。

    如果`audit_log_rotate_on_size`大于 0，则会发生基于大小的自动日志文件轮换。每当写入日志文件导致其大小超过`audit_log_rotate_on_size`值时，审计日志插件会重命名当前日志文件，并使用原始名称打开一个新的当前日志文件。

    如果将`audit_log_rotate_on_size`设置为不是 4096 的倍数的值，则会将其截断为最接近的倍数。特别是，将其设置为小于 4096 的值会将其设置为 0，并且不会发生旋转，除非手动执行。

    注意

    `audit_log_rotate_on_size`控制是否发生审计日志文件轮换。它还可以与`audit_log_max_size`和`audit_log_prune_seconds`一起用于配置旋转 JSON 格式日志文件的修剪。有关详细信息，请参见审计日志文件的空间管理。

+   `audit_log_statement_policy`

    | 命令行格式 | `--audit-log-statement-policy=value` |
    | --- | --- |
    | 已弃用 | 8.0.34 |
    | 系统变量 | `audit_log_statement_policy` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `ALL` |
    | 有效值 | `ALL``ERRORS``NONE` |

    注意

    此已弃用的变量仅适用于传统模式审计日志过滤（参见第 8.4.5.10 节，“传统模式审计日志过滤”）。

    控制审计日志插件如何将语句事件写入其日志文件的策略。以下表格显示了允许的值。

    | 值 | 描述 |
    | --- | --- |
    | `ALL` | 记录所有语句事件 |
    | `ERRORS` | 仅记录失败的语句事件 |
    | `NONE` | 不记录语句事件 |

    注意

    在服务器启动时，如果还指定了`audit_log_policy`，则可以覆盖为`audit_log_statement_policy`给出的任何显式值，如第 8.4.5.5 节，“配置审计日志特性”中所述。

+   `audit_log_strategy`

    | 命令行格式 | `--audit-log-strategy=value` |
    | --- | --- |
    | 系统变量 | `audit_log_strategy` |
    | 范围 | 全局 |
    | 动态 | 否 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 枚举 |
    | 默认值 | `ASYNCHRONOUS` |
    | 有效值 | `ASYNCHRONOUS``PERFORMANCE``SEMISYNCHRONOUS``SYNCHRONOUS` |

    审计日志插件使用的记录方法。允许使用以下策略值：

    +   `ASYNCHRONOUS`：异步记录。等待输出缓冲区中的空间。

    +   `PERFORMANCE`：异步记录。在输出缓冲区中没有足够空间的请求将被丢弃。

    +   `SEMISYNCHRONOUS`：同步记录。允许操作系统进行缓存。

    +   `SYNCHRONOUS`：同步记录。在每个请求后调用`sync()`。

##### 审计日志状态变量

如果启用了审计日志插件，则会公开几个提供操作信息的状态变量。这些变量适用于遗留模式审计过滤（在 MySQL 8.0.34 中已弃用）和 JSON 模式审计过滤。

+   `Audit_log_current_size`

    当前审计日志文件的大小。当事件写入日志时，该值会增加，并在日志轮换时重置为 0。

+   `Audit_log_event_max_drop_size`

    性能记录模式中最大丢弃事件的大小。有关记录模式的描述，请参见第 8.4.5.5 节，“配置审计日志特性”。

+   `Audit_log_events`

    由审计日志插件处理的事件数量，无论是否根据过滤策略写入日志（参见第 8.4.5.5 节，“配置审计日志特性”）。

+   `Audit_log_events_filtered`

    基于过滤策略（参见第 8.4.5.5 节，“配置审计日志特性”），由审计日志插件处理的事件数量被过滤（未写入日志）。

+   `Audit_log_events_lost`

    在性能记录模式下丢失的事件数量，因为事件大于可用的审计日志缓冲区空间。此值可能有助于评估如何设置`audit_log_buffer_size`以调整性能模式的缓冲区大小。有关记录模式的描述，请参见第 8.4.5.5 节，“配置审计日志特性”。

+   `Audit_log_events_written`

    写入审计日志的事件数量。

+   `Audit_log_total_size`

    所有审计日志文件中写入的事件总大小。与`Audit_log_current_size`不同，`Audit_log_total_size`的值在日志轮换时也会增加。

+   `Audit_log_write_waits`

    在异步记录模式下，事件必须等待审计日志缓冲区中的空间的次数。有关记录模式的描述，请参见第 8.4.5.5 节，“配置审计日志特性”。
