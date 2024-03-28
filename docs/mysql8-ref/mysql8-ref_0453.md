> 原文：[`dev.mysql.com/doc/refman/8.0/en/firewall-reference.html`](https://dev.mysql.com/doc/refman/8.0/en/firewall-reference.html)

#### 8.4.7.4 MySQL 企业防火墙参考

以下各节提供了 MySQL 企业防火墙元素的参考：

+   MySQL 企业防火墙表

+   MySQL 企业防火墙存储过程

+   MySQL 企业防火墙管理功能

+   MySQL 企业防火墙系统变量

+   MySQL 企业防火墙状态变量

##### MySQL 企业防火墙表

MySQL 企业防火墙根据组和帐户的基础信息维护配置文件信息。它使用`mysql`系统数据库中的表进行持久存储，并使用`INFORMATION_SCHEMA`或 Performance Schema 表提供对内存中缓存数据的视图。启用后，防火墙基于缓存数据做出操作决策。

+   防火墙组配置文件表

+   防火墙帐户配置文件表

###### 防火墙组配置文件表

截至 MySQL 8.0.23，MySQL 企业防火墙使用`mysql`系统数据库中的表来维护组配置文件信息以进行持久存储，并使用 Performance Schema 表提供对内存中缓存数据的视图。

每个系统和 Performance Schema 表只能由具有相应`SELECT`权限的帐户访问。

`mysql.firewall_groups`表列出了注册的防火墙组配置文件的名称和操作模式。该表具有以下列（相应的 Performance Schema `firewall_groups`表具有类似但不一定相同的列）：

+   `NAME`

    组配置文件名称。

+   `MODE`

    配置文件的当前操作模式。允许的模式值为`OFF`、`DETECTING`、`PROTECTING`和`RECORDING`。有关它们含义的详细信息，请参见防火墙概念。

+   `USERHOST`

    组配置文件的训练帐户，在配置文件处于`RECORDING`模式时使用。该值为`NULL`，或具有格式`*`user_name`*@*`host_name`*`的非`NULL`帐户：

    +   如果值为`NULL`，防火墙将记录来自组成员的任何帐户接收到的 allowlist 规则。

    +   如果值为非`NULL`，防火墙仅记录来自指定帐户（应为组的成员）接收的语句的允许列表规则。

`mysql.firewall_group_allowlist`表列出了注册的防火墙组配置文件的允许列表规则。该表具有以下列（相应的性能模式`firewall_group_allowlist`表具有类似但不一定相同的列）：

+   `NAME`

    组配置文件名称。

+   `RULE`

    表示配置文件可接受语句模式的规范化语句。配置文件允许列表是其规则的并集。

+   `ID`

    用于表的主键的整数列。

`mysql.firewall_membership`表列出了注册的防火墙组配置文件的成员（帐户）。该表具有以下列（相应的性能模式`firewall_membership`表具有类似但不一定相同的列）：

+   `GROUP_ID`

    组配置文件名称。

+   `MEMBER_ID`

    是配置文件成员的帐户名称。

###### 防火墙帐户配置表

MySQL 企业防火墙使用`mysql`系统数据库中的表进行持久存储的帐户配置文件信息，并使用`INFORMATION_SCHEMA`表提供对内存中缓存数据的视图。

每个`mysql`系统数据库表只能被具有相应`SELECT`权限的帐户访问。`INFORMATION_SCHEMA`表可被任何人访问。

截至 MySQL 8.0.26，这些表已被弃用，并可能在未来的 MySQL 版本中被移除。请参阅 Migrating Account Profiles to Group Profiles。

`mysql.firewall_users`表列出了注册的防火墙帐户配置文件的名称和操作模式。该表具有以下列（相应的`MYSQL_FIREWALL_USERS`表具有类似但不一定相同的列）：

+   `USERHOST`

    帐户配置文件名称。每个帐户名称的格式为`*`user_name`*@*`host_name`*`。

+   `MODE`

    配置文件的当前操作模式。允许的模式值为`OFF`、`DETECTING`、`PROTECTING`、`RECORDING`和`RESET`。有关其含义的详细信息，请参阅 Firewall Concepts。

`mysql.firewall_whitelist`表列出了已注册的防火墙帐户配置文件的允许列表规则。该表具有以下列（相应的`MYSQL_FIREWALL_WHITELIST`表具有类似但不一定相同的列）:

+   `USERHOST`

    帐户配置文件名称。每个帐户名称的格式为`*`user_name`*@*`host_name`*`。

+   `RULE`

    表示配置文件的可接受语句模式的规范化语句。配置文件的允许列表是其规则的并集。

+   `ID`

    表的主键是整数列。此列在 MySQL 8.0.12 中添加。

##### MySQL 企业防火墙存储过程

MySQL 企业防火墙存储过程执行诸如向防火墙注册配置文件、建立其操作模式以及管理防火墙数据在缓存和持久存储之间传输等任务。这些存储过程调用提供用于低级任务的 API 的管理函数。

防火墙存储过程在`mysql`系统数据库中创建。要调用防火墙存储过程，可以在`mysql`是默认数据库时执行，或者使用数据库名称限定过程名称。例如:

```sql
CALL mysql.sp_set_firewall_group_mode(*group*, *mode*);
```

+   防火墙组配置文件存储过程

+   防火墙帐户配置文件存储过程

+   防火墙杂项存储过程

###### 防火墙组配置文件存储过程

这些存储过程在防火墙组配置文件上执行管理操作:

+   `sp_firewall_group_delist(*`group`*, *`user`*)`

    此存储过程从防火墙组配置文件中删除一个帐户。

    如果调用成功，则组成员身份的更改将同时应用于内存缓存和持久存储。

    参数:

    +   *`group`*: 受影响的组配置文件的名称。

    +   *`user`*: 要移除的帐户，格式为`*`user_name`*@*`host_name`*`。

    例子:

    ```sql
    CALL sp_firewall_group_delist('g', 'fwuser@localhost');
    ```

    此存储过程在 MySQL 8.0.23 中添加。

+   `sp_firewall_group_enlist(*`group`*, *`user`*)`

    此存储过程将一个帐户添加到防火墙组配置文件中。在将帐户添加到组之前，不需要将帐户本身注册到防火墙。

    如果调用成功，则组成员身份的更改将同时应用于内存缓存和持久存储。

    参数:

    +   *`group`*: 受影响的组配置文件的名称。

    +   *`user`*: 要添加的帐户，格式为`*`user_name`*@*`host_name`*`。

    例子:

    ```sql
    CALL sp_firewall_group_enlist('g', 'fwuser@localhost');
    ```

    此存储过程在 MySQL 8.0.23 中添加。

+   `sp_reload_firewall_group_rules(*`group`*)`

    此存储过程为各个组配置文件提供防火墙操作控制。该过程使用防火墙管理函数从`mysql.firewall_group_allowlist`表中重新加载组配置文件的内存规则。

    参数：

    +   *`group`*：受影响的组配置文件的名称。

    示例：

    ```sql
    CALL sp_reload_firewall_group_rules('myapp');
    ```

    警告

    此存储过程在重新加载持久存储中的组配置文件的允许规则之前清除组配置文件的内存中的允许规则，并将配置文件模式设置为`OFF`。如果在调用`sp_reload_firewall_group_rules()`之前配置文件模式不是`OFF`，则在重新加载规则后使用`sp_set_firewall_group_mode()`将其恢复为先前的模式。例如，如果配置文件处于`PROTECTING`模式，则在调用`sp_reload_firewall_group_rules()`后不再有效，必须显式将其设置为`PROTECTING`。

    此存储过程在 MySQL 8.0.23 中添加。

+   `sp_set_firewall_group_mode(*`group`*, *`mode`*)`

    此存储过程在将配置文件注册到防火墙后，为防火墙组配置文件建立操作模式，如果尚未注册，则调用防火墙管理函数以在缓存和持久存储之间传输防火墙数据。即使`mysql_firewall_mode`系统变量为`OFF`，也可以调用此存储过程，尽管在启用防火墙之前，为配置文件设置模式不会产生操作效果。

    如果配置文件之前存在，则其任何记录限制保持不变。要设置或清除限制，请改为调用`sp_set_firewall_group_mode_and_user()`。

    参数：

    +   *`group`*：受影响的组配置文件的名称。

    +   *`mode`*：配置文件的操作模式，以字符串形式表示。允许的模式值为`OFF`、`DETECTING`、`PROTECTING`和`RECORDING`。有关它们的含义的详细信息，请参见防火墙概念。

    示例：

    ```sql
    CALL sp_set_firewall_group_mode('myapp', 'PROTECTING');
    ```

    此存储过程在 MySQL 8.0.23 中添加。

+   `sp_set_firewall_group_mode_and_user(*`group`*, *`mode`*, *`user`*)`

    此存储过程向防火墙注册一个组并建立其操作模式，类似于`sp_set_firewall_group_mode()`，但还指定了在组处于`RECORDING`模式时要使用的训练帐户。

    参数：

    +   *`group`*：受影响的组配置文件的名称。

    +   *`mode`*：配置文件的操作模式，以字符串形式表示。允许的模式值为`OFF`、`DETECTING`、`PROTECTING`和`RECORDING`。有关它们的含义的详细信息，请参见防火墙概念。

    +   *`user`*：组配置文件的训练帐户，在配置文件处于`RECORDING`模式时使用。该值为`NULL`，或具有格式`*`user_name`*@*`host_name`*`的非`NULL`帐户：

        +   如果数值为`NULL`，防火墙将记录来自任何属于该组的帐户接收到的语句的允许规则。

        +   如果值为非`NULL`，则防火墙记录仅允许来自指定账户（应为组的成员）的语句。

    示例：

    ```sql
    CALL sp_set_firewall_group_mode_and_user('myapp', 'RECORDING', 'myapp_user1@localhost');
    ```

    此过程是在 MySQL 8.0.23 中添加的。

###### 防火墙账户配置文件存储过程

这些存储过程执行防火墙账户配置文件的管理操作：

+   `sp_reload_firewall_rules(*`user`*)`

    此存储过程为个别账户配置文件提供防火墙操作控制。该过程使用防火墙管理函数从`mysql.firewall_whitelist`表中存储的规则重新加载账户配置文件的内存中规则。

    参数：

    +   *`user`*: 受影响的账户配置文件的名称，以`*`user_name`*@*`host_name`*`格式的字符串表示。

    示例：

    ```sql
    CALL mysql.sp_reload_firewall_rules('fwuser@localhost');
    ```

    警告

    此过程在重新加载持久存储中的账户配置文件内存中的允许列表规则之前清除它们，并将配置文件模式设置为`OFF`。如果在调用`sp_reload_firewall_rules()`之前配置文件模式不是`OFF`，则在重新加载规则后使用`sp_set_firewall_mode()`将其恢复为先前的模式。例如，如果配置文件处于`PROTECTING`模式，则在调用`sp_reload_firewall_rules()`后不再有效，必须显式将其设置为`PROTECTING`。

    从 MySQL 8.0.26 开始，此存储过程已弃用，并可能在将来的 MySQL 版本中删除。请参见将账户配置文件迁移到组配置文件。

+   `sp_set_firewall_mode(*`user`*, *`mode`*)`

    此存储过程在注册配置文件到防火墙后，为防火墙账户配置文件建立操作模式。必要时，该过程还调用防火墙管理函数以在缓存和持久存储之间传输防火墙数据。即使`mysql_firewall_mode`系统变量为`OFF`，也可以调用此过程，尽管在启用防火墙之前，为配置文件设置模式不会产生操作效果。

    参数：

    +   *`user`*: 受影响的账户配置文件的名称，以`*`user_name`*@*`host_name`*`格式的字符串表示。

    +   *`mode`*: 该配置文件的操作模式，以字符串形式表示。允许的模式值为`OFF`、`DETECTING`、`PROTECTING`、`RECORDING`和`RESET`。有关它们含义的详细信息，请参见防火墙概念。

    将账户配置文件切换到任何模式，但`RECORDING`会将其防火墙缓存数据同步到提供持久底层存储的`mysql`系统数据库表中。将模式从`OFF`切换到`RECORDING`会将允许列表从`mysql.firewall_whitelist`表重新加载到缓存中。

    如果一个账户配置具有空的允许列表，其模式不能设置为`PROTECTING`，因为该配置将拒绝每个语句，实际上会禁止该账户执行语句。针对这种模式设置尝试，防火墙会生成一条诊断消息，该消息作为结果集返回，而不是作为 SQL 错误：

    ```sql
    mysql> CALL mysql.sp_set_firewall_mode('a@b','PROTECTING');
    +----------------------------------------------------------------------+
    | set_firewall_mode(arg_userhost, arg_mode)                            |
    +----------------------------------------------------------------------+
    | ERROR: PROTECTING mode requested for a@b but the allowlist is empty. |
    +----------------------------------------------------------------------+
    ```

    从 MySQL 8.0.26 开始，此过程已被弃用，并可能在未来的 MySQL 版本中移除。请参阅 Migrating Account Profiles to Group Profiles。

###### 防火墙杂项存储过程

这些存储过程执行杂项防火墙管理操作。

+   `sp_migrate_firewall_user_to_group(*`user`*, *`group`*)`

    从 MySQL 8.0.26 开始，账户配置已被弃用，因为组配置可以执行账户配置可以执行的任何操作。`sp_migrate_firewall_user_to_group()`存储过程将防火墙账户配置转换为以该账户为唯一成员的组配置。运行`firewall_profile_migration.sql`脚本进行安装。转换过程在 Migrating Account Profiles to Group Profiles 中讨论。

    此例程需要`FIREWALL_ADMIN`权限。

    参数：

    +   *`user`*: 要转换为组配置的账户配置的名称，以`*`user_name`*@*`host_name`*`格式的字符串。账户配置必须存在，并且不能当前处于`RECORDING`模式。

    +   *`group`*: 新组配置的名称，该名称不能已存在。新组配置以命名账户作为其唯一成员，并且该成员被设置为组培训账户。组配置操作模式取自账户配置操作模式。

    示例：

    ```sql
    CALL sp_migrate_firewall_user_to_group('fwuser@localhost', 'mygroup);
    ```

    此过程已添加到 MySQL 8.0.26 中。

##### MySQL 企业防火墙管理功能

MySQL 企业防火墙管理功能提供了一个 API，用于执行诸如将防火墙缓存与底层系统表同步等较低级别任务。

*在正常操作中，这些功能由防火墙存储过程调用，而不是直接由用户调用。*因此，这些功能描述不包括有关其参数和返回类型的详细信息。

+   防火墙组配置功能

+   防火墙账户配置功能

+   防火墙杂项功能

###### 防火墙组配置功能

这些功能执行防火墙组配置的管理操作：

+   `firewall_group_delist(*`group`*, *`user`*)`

    此函数从组配置文件中删除帐户。它需要`FIREWALL_ADMIN`权限。

    示例：

    ```sql
    SELECT firewall_group_delist('g', 'fwuser@localhost');
    ```

    此函数在 MySQL 8.0.23 中添加。

+   `firewall_group_enlist(*`group`*, *`user`*)`

    此函数将帐户添加到组配置文件中。它需要`FIREWALL_ADMIN`权限。

    在将帐户添加到组之前，不需要将帐户本身注册到防火墙。

    示例：

    ```sql
    SELECT firewall_group_enlist('g', 'fwuser@localhost');
    ```

    此函数在 MySQL 8.0.23 中添加。

+   `read_firewall_group_allowlist(*`group`*, *`rule`*)`

    此聚合函数通过对`mysql.firewall_group_allowlist`表上的`SELECT`语句更新了命名组配置文件的记录语句缓存。它需要`FIREWALL_ADMIN`权限。

    示例：

    ```sql
    SELECT read_firewall_group_allowlist('my_fw_group', fgw.rule)
    FROM mysql.firewall_group_allowlist AS fgw
    WHERE NAME = 'my_fw_group';
    ```

    此函数在 MySQL 8.0.23 中添加。

+   `read_firewall_groups(*`group`*, *`mode`*, *`user`*)`

    此聚合函数通过对`mysql.firewall_groups`表上的`SELECT`语句更新了防火墙组配置文件缓存。它需要`FIREWALL_ADMIN`权限。

    示例：

    ```sql
    SELECT read_firewall_groups('g', 'RECORDING', 'fwuser@localhost')
    FROM mysql.firewall_groups;
    ```

    此函数在 MySQL 8.0.23 中添加。

+   [`set_firewall_group_mode(*`group`*, *`mode`*[, *`user`*])`](firewall-reference.html#function_set-firewall-group-mode)

    此函数管理组配置文件缓存，建立配置文件操作模式，并可选择指定配置文件训练帐户。它需要`FIREWALL_ADMIN`权限。

    如果未提供可选的*`user`*参数，则配置文件的先前*`user`*设置保持不变。要更改设置，请使用第三个参数调用该函数。

    如果提供了可选的*`user`*参数，则指定组配置文件的训练帐户，用于配置文件处于`RECORDING`模式时使用。该值为`NULL`，或具有格式`*`user_name`*@*`host_name`*`的非`NULL`帐户：

    +   如果值为`NULL`，防火墙记录允许列表规则用于接收来自任何属于该组的帐户的语句。

    +   如果值为非`NULL`，则防火墙仅记录来自命名帐户（应为组成员）的语句的允许列表规则。

    示例：

    ```sql
    SELECT set_firewall_group_mode('g', 'DETECTING');
    ```

    此函数在 MySQL 8.0.23 中添加。

###### 防火墙帐户配置文件函数

这些函数执行防火墙帐户配置文件的管理操作：

+   `read_firewall_users(*`user`*, *`mode`*)`

    此聚合函数通过对`mysql.firewall_users`表上的`SELECT`语句更新防火墙帐户配置文件缓存。它需要`FIREWALL_ADMIN`权限或已弃用的`SUPER`权限。

    示例:

    ```sql
    SELECT read_firewall_users('fwuser@localhost', 'RECORDING')
    FROM mysql.firewall_users;
    ```

    截至 MySQL 8.0.26 版本，此函数已被弃用，并可能在未来的 MySQL 版本中被移除。请参阅将帐户配置文件迁移到组配置文件。

+   `read_firewall_whitelist(*`user`*, *`rule`*)`

    此聚合函数通过对`mysql.firewall_whitelist`表上的`SELECT`语句更新命名帐户配置文件的记录语句缓存。它需要`FIREWALL_ADMIN`权限或已弃用的`SUPER`权限。

    示例:

    ```sql
    SELECT read_firewall_whitelist('fwuser@localhost', fw.rule)
    FROM mysql.firewall_whitelist AS fw
    WHERE USERHOST = 'fwuser@localhost';
    ```

    截至 MySQL 8.0.26 版本，此函数已被弃用，并可能在未来的 MySQL 版本中被移除。请参阅将帐户配置文件迁移到组配置文件。

+   `set_firewall_mode(*`user`*, *`mode`*)`

    此函数管理帐户配置文件缓存并建立配置文件操作模式。它需要`FIREWALL_ADMIN`权限或已弃用的`SUPER`权限。

    示例:

    ```sql
    SELECT set_firewall_mode('fwuser@localhost', 'RECORDING');
    ```

    截至 MySQL 8.0.26 版本，此函数已被弃用，并可能在未来的 MySQL 版本中被移除。请参阅将帐户配置文件迁移到组配置文件。

###### 防火墙杂项函数

这些函数执行各种防火墙操作:

+   `mysql_firewall_flush_status()`

    此函数将多个防火墙状态变量重置为 0：

    +   `Firewall_access_denied`

    +   `Firewall_access_granted`

    +   `Firewall_access_suspicious`

    此函数需要`FIREWALL_ADMIN`权限或已弃用的`SUPER`权限。

    示例:

    ```sql
    SELECT mysql_firewall_flush_status();
    ```

+   `normalize_statement(*`stmt`*)`

    此函数将 SQL 语句规范化为用于允许列表规则的摘要形式。它需要`FIREWALL_ADMIN`权限或已弃用的`SUPER`权限。

    示例:

    ```sql
    SELECT normalize_statement('SELECT * FROM t1 WHERE c1 > 2');
    ```

    注意

    相同的摘要功能在防火墙上下文之外使用`STATEMENT_DIGEST_TEXT()` SQL 函数可用。

##### MySQL 企业防火墙系统变量

MySQL 企业防火墙支持以下系统变量。使用它们来配置防火墙操作。除非安装了防火墙（参见第 8.4.7.2 节，“安装或卸载 MySQL 企业防火墙”），否则这些变量不可用。

+   `mysql_firewall_mode`

    | 命令行格式 | `--mysql-firewall-mode[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `mysql_firewall_mode` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `ON` |

    MySQL 企业防火墙是否启用（默认情况下）或禁用。

+   `mysql_firewall_trace`

    | 命令行格式 | `--mysql-firewall-trace[={OFF&#124;ON}]` |
    | --- | --- |
    | 系统变量 | `mysql_firewall_trace` |
    | 范围 | 全局 |
    | 动态 | 是 |
    | `SET_VAR`提示适用 | 否 |
    | 类型 | 布尔值 |
    | 默认值 | `OFF` |

    MySQL 企业防火墙跟踪是否启用或禁用（默认情况下）。当`mysql_firewall_trace`启用时，在`PROTECTING`模式下，防火墙会将被拒绝的语句写入错误日志。

##### MySQL 企业防火墙状态变量

MySQL 企业防火墙支持以下状态变量。使用它们来获取有关防火墙运行状态的信息。除非安装了防火墙（参见第 8.4.7.2 节，“安装或卸载 MySQL 企业防火墙”），否则这些变量不可用。防火墙状态变量在安装`MYSQL_FIREWALL`插件或启动服务器时被设置为 0。许多变量通过`mysql_firewall_flush_status()`函数重置为零（参见 MySQL 企业防火墙管理函数）。

+   `Firewall_access_denied`

    被 MySQL 企业防火墙拒绝的语句数量。

+   `Firewall_access_granted`

    被 MySQL 企业防火墙接受的语句数量。

+   `Firewall_access_suspicious`

    MySQL 企业防火墙记录为可疑的语句数量，适用于处于`DETECTING`模式的用户。

+   `Firewall_cached_entries`

    MySQL 企业防火墙记录的语句数量，包括重复的语句。
