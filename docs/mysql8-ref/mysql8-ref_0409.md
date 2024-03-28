> 原文：[`dev.mysql.com/doc/refman/8.0/en/connection-control-installation.html`](https://dev.mysql.com/doc/refman/8.0/en/connection-control-installation.html)

#### 8.4.2.1 连接控制插件安装

本节描述了如何安装连接控制插件`CONNECTION_CONTROL`和`CONNECTION_CONTROL_FAILED_LOGIN_ATTEMPTS`。有关安装插件的一般信息，请参见第 7.6.1 节，“安装和卸载插件”。

要使服务器可用，插件库文件必须位于 MySQL 插件目录中（由`plugin_dir`系统变量命名的目录）。如果需要，通过在服务器启动时设置`plugin_dir`的值来配置插件目录位置。

插件库文件的基本名称是`connection_control`。文件名后缀因平台而异（例如，对于 Unix 和类 Unix 系统，为`.so`，对于 Windows 为`.dll`）。

要在服务器启动时加载插件，请使用`--plugin-load-add`选项命名包含它们的库文件。使用此插件加载方法，选项必须在每次服务器启动时给出。例如，将以下行放入服务器的`my.cnf`文件中，根据需要调整您平台的`.so`后缀：

```sql
[mysqld]
plugin-load-add=connection_control.so
```

修改`my.cnf`后，重新启动服务器以使新设置生效。

或者，要在运行时加载插件，请使用以下语句，根据需要调整您平台的`.so`后缀：

```sql
INSTALL PLUGIN CONNECTION_CONTROL
  SONAME 'connection_control.so';
INSTALL PLUGIN CONNECTION_CONTROL_FAILED_LOGIN_ATTEMPTS
  SONAME 'connection_control.so';
```

`INSTALL PLUGIN`立即加载插件，并在`mysql.plugins`系统表中注册它，以使服务器在每次后续正常启动时加载它，而无需`--plugin-load-add`。

要验证插件安装，请检查信息模式`PLUGINS`表，或使用`SHOW PLUGINS`语句（参见第 7.6.2 节，“获取服务器插件信息”）。例如：

```sql
mysql> SELECT PLUGIN_NAME, PLUGIN_STATUS
       FROM INFORMATION_SCHEMA.PLUGINS
       WHERE PLUGIN_NAME LIKE 'connection%';
+------------------------------------------+---------------+
| PLUGIN_NAME                              | PLUGIN_STATUS |
+------------------------------------------+---------------+
| CONNECTION_CONTROL                       | ACTIVE        |
| CONNECTION_CONTROL_FAILED_LOGIN_ATTEMPTS | ACTIVE        |
+------------------------------------------+---------------+
```

如果插件初始化失败，请检查服务器错误日志以获取诊断信息。

如果插件之前已经使用`INSTALL PLUGIN`注册过，或者使用`--plugin-load-add`加载过，您可以在服务器启动时使用`--connection-control`和`--connection-control-failed-login-attempts`选项来控制插件的激活。例如，要在启动时加载插件并防止它们在运行时被移除，请使用以下选项：

```sql
[mysqld]
plugin-load-add=connection_control.so
connection-control=FORCE_PLUS_PERMANENT
connection-control-failed-login-attempts=FORCE_PLUS_PERMANENT
```

如果希望防止服务器在没有给定连接控制插件的情况下运行，请使用 `FORCE` 或 `FORCE_PLUS_PERMANENT` 的选项值，以强制服务器启动失败，如果插件未成功初始化。

注意

可以安装一个插件而不安装另一个插件，但必须同时安装两个插件才能实现完整的连接控制功能。特别是，仅安装 `CONNECTION_CONTROL_FAILED_LOGIN_ATTEMPTS` 插件几乎没有用处，因为没有 `CONNECTION_CONTROL` 插件提供数据来填充 `CONNECTION_CONTROL_FAILED_LOGIN_ATTEMPTS` 表，该表始终为空。

+   连接延迟配置

+   连接失败评估

+   连接失败监控

##### 连接延迟配置

要启用其操作配置，`CONNECTION_CONTROL` 插件公开了这些系统变量：

+   `connection_control_failed_connections_threshold`: 在服务器在为账户添加延迟以进行后续连接尝试之前允许的连续失败连接尝试次数。要禁用失败连接计数，请将 `connection_control_failed_connections_threshold` 设置为零。

+   `connection_control_min_connection_delay`: 连接失败的最小延迟时间（毫秒）超过阈值。

+   `connection_control_max_connection_delay`: 连接失败的最大延迟时间（毫秒）超过阈值。

如果 `connection_control_failed_connections_threshold` 不为零，则启用了失败连接计数，并具有以下属性：

+   延迟为零，直到`connection_control_failed_connections_threshold` 连续失败的连接尝试。

+   之后，服务器会为后续连续尝试添加递增延迟，直到成功连接发生。初始未调整的延迟从 1000 毫秒（1 秒）开始，每次尝试增加 1000 毫秒。也就是说，一旦账户激活延迟，后续失败尝试的未调整延迟为 1000 毫秒、2000 毫秒、3000 毫秒等。

+   客户端实际经历的延迟是未调整的延迟，调整后的值在`connection_control_min_connection_delay`和`connection_control_max_connection_delay`系统变量的值范围内。

+   一旦账户激活延迟，随后该账户的第一次成功连接也会经历延迟，但是对于后续连接，失败计数会被重置。

例如，使用默认的`connection_control_failed_connections_threshold`值为 3，账户的前三次连续连接尝试失败不会有延迟。账户在第四次及后续失败连接中实际调整的延迟取决于`connection_control_min_connection_delay`和`connection_control_max_connection_delay`的值：

+   如果`connection_control_min_connection_delay`和`connection_control_max_connection_delay`分别为 1000 和 20000，调整后的延迟与未调整的延迟相同，最多不超过 20000 毫秒。第四次及后续失败连接延迟为 1000 毫秒、2000 毫秒、3000 毫秒等。

+   如果`connection_control_min_connection_delay`和`connection_control_max_connection_delay`分别为 1500 和 20000，第四次及后续失败连接的调整延迟为 1500 毫秒、2000 毫秒、3000 毫秒等，最多不超过 20000 毫秒。

+   如果 `connection_control_min_connection_delay` 和 `connection_control_max_connection_delay` 分别为 2000 和 3000，则第四次及后续失败连接的调整延迟为 2000 毫秒、2000 毫秒和 3000 毫秒，所有后续失败连接也将延迟 3000 毫秒。

您可以在服务器启动或运行时设置`CONNECTION_CONTROL`系统变量。假设您希望在服务器开始延迟其响应之前允许四次连续失败的连接尝试，最小延迟为 2000 毫秒。要在服务器启动时设置相关变量，请将以下行放入服务器的`my.cnf`文件中：

```sql
[mysqld]
plugin-load-add=connection_control.so
connection_control_failed_connections_threshold=4
connection_control_min_connection_delay=2000
```

要在运行时设置并持久化变量，请使用以下语句：

```sql
SET PERSIST connection_control_failed_connections_threshold = 4;
SET PERSIST connection_control_min_connection_delay = 2000;
```

`SET PERSIST` 为正在运行的 MySQL 实例设置一个值。它还保存该值，导致其在后续服务器重新启动时保留。要更改正在运行的 MySQL 实例的值，而不使其在后续重新启动时保留，使用`GLOBAL`关键字而不是`PERSIST`。参见 Section 15.7.6.1, “变量赋值的 SET 语法”。

`connection_control_min_connection_delay` 和 `connection_control_max_connection_delay` 系统变量的最小值和最大值均为 1000 和 2147483647。此外，每个变量的允许值范围还取决于另一个变量的当前值：

+   `connection_control_min_connection_delay` 不能设置为大于当前值`connection_control_max_connection_delay`。

+   `connection_control_max_connection_delay` 不能设置为小于当前值`connection_control_min_connection_delay`。

因此，为了对某些配置所需的更改进行设置，您可能需要按特定顺序设置变量。假设当前的最小延迟和最大延迟分别为 1000 和 2000，并且您想将它们设置为 3000 和 5000。您不能首先将`connection_control_min_connection_delay`设置为 3000，因为这大于当前的`connection_control_max_connection_delay`值 2000。相反，先将`connection_control_max_connection_delay`设置为 5000，然后将`connection_control_min_connection_delay`设置为 3000。

##### 连接失败评估

当安装了`CONNECTION_CONTROL`插件时，它会检查连接尝试并跟踪它们是失败还是成功。为此，连接失败尝试是指客户端用户和主机匹配已知的 MySQL 账户，但提供的凭据不正确，或者不匹配任何已知账户。

失败连接计数基于每次连接尝试的用户/主机组合。适用用户名称和主机名称的确定考虑了代理，并按以下方式进行：

+   如果客户端用户代理另一个用户，则失败连接计数的账户是代理用户，而不是被代理用户。例如，如果`external_user@example.com`代理了`proxy_user@example.com`，连接计数使用代理用户`external_user@example.com`，而不是被代理用户`proxy_user@example.com`。`external_user@example.com`和`proxy_user@example.com`必须在`mysql.user`系统表中有有效条目，并且它们之间必须在`mysql.proxies_priv`系统表中定义代理关系（参见第 8.2.19 节，“代理用户”）。

+   如果客户端用户没有代理另一个用户，但匹配了一个`mysql.user`条目，则计数使用与该条目对应的`CURRENT_USER()`值。例如，如果用户`user1`从主机`host1.example.com`连接匹配了`user1@host1.example.com`条目，则计数使用`user1@host1.example.com`。如果用户匹配了`user1@%.example.com`、`user1@%.com`或`user1@%`条目，计数分别使用`user1@%.example.com`、`user1@%.com`或`user1@%`。

对于刚才描述的情况，连接尝试匹配了一些`mysql.user`条目，请求成功或失败取决于客户端是否提供了正确的身份验证凭据。例如，如果客户端提供了错误的密码，连接尝试将失败。

如果连接尝试与任何 `mysql.user` 条目不匹配，则尝试失败。在这种情况下，没有 `CURRENT_USER()` 值可用，连接失败计数使用客户端提供的用户名和服务器确定的客户端主机。例如，如果客户端尝试作为用户 `user2` 从主机 `host2.example.com` 连接，则用户名称部分在客户端请求中可用，服务器确定主机信息。用于计数的用户/主机组合是 `user2@host2.example.com`。

注意

服务器维护关于哪些客户端主机可能连接到服务器的信息（基本上是 `mysql.user` 条目的主机值的并集）。如果客户端尝试从任何其他主机连接，服务器在连接设置的早期阶段拒绝尝试：

```sql
ERROR 1130 (HY000): Host '*host_name*' is not
allowed to connect to this MySQL server
```

因为这种类型的拒绝发生得如此早，`CONNECTION_CONTROL` 看不到它，并且不计数。

##### 连接失败监控

要监视失败连接，请使用以下信息来源：

+   `Connection_control_delay_generated` 状态变量指示服务器在响应失败连接尝试时添加延迟的次数。这不包括在达到由 `connection_control_failed_connections_threshold` 系统变量定义的阈值之前发生的尝试。

+   `INFORMATION_SCHEMA` `CONNECTION_CONTROL_FAILED_LOGIN_ATTEMPTS` 表提供了关于每个账户（用户/主机组合）连续失败连接尝试的当前数量的信息。这计算所有失败尝试，无论它们是否被延迟。

在运行时为 `connection_control_failed_connections_threshold` 赋值会产生以下效果：

+   所有累积的失败连接计数器被重置为零。

+   `Connection_control_delay_generated` 状态变量被重置为零。

+   `CONNECTION_CONTROL_FAILED_LOGIN_ATTEMPTS` 表变为空。
