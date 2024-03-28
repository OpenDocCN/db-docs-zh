> 原文：[`dev.mysql.com/doc/refman/8.0/en/audit-log-filtering.html`](https://dev.mysql.com/doc/refman/8.0/en/audit-log-filtering.html)

#### 8.4.5.7 审计日志过滤

注意

要使此处描述的审计日志过滤工作，必须安装审计日志插件*和相应的审计表和函数*。如果安装了插件但没有为基于规则的过滤所需的审计表和函数，插件将以传统过滤模式运行，描述在第 8.4.5.10 节“传统模式审计日志过滤”中。传统模式（在 MySQL 8.0.34 中已弃用）是 MySQL 5.7.13 之前引入基于规则的过滤之前的过滤行为。

+   审计日志过滤属性

+   审计日志过滤函数的约束

+   使用审计日志过滤函数

##### 审计日志过滤属性

审计日志插件具有通过过滤控制记录审计事件的能力：

+   可以使用以下特征过滤审计事件：

    +   用户帐户

    +   审计事件类

    +   审计事件子类

    +   诸如指示操作状态或执行的 SQL 语句等审计事件字段

+   审计过滤是基于规则的：

    +   审计规则定义创建了一组审计规则。根据刚才描述的特征，可以配置定义以包括或排除事件进行日志记录。

    +   过滤规则具有阻止（中止）符合条件事件执行的能力，除了现有的事件记录功能之外。

    +   可以定义多个过滤器，并且任何给定的过滤器可以分配给任意数量的用户帐户。

    +   可以定义一个默认过滤器，用于任何没有明确分配过滤器的用户帐户。

    审计日志过滤用于从 MySQL 8.0.30 实现组件服务。要获取该版本提供的可选查询统计信息，您可以设置它们作为一个使用服务组件的过滤器，该组件实现将统计信息写入审计日志的服务。有关设置此过滤器的说明，请参见添加查询统计信息以进行异常检测。

    有关编写过滤规则的信息，请参见第 8.4.5.8 节“编写审计日志过滤器定义”。

+   可以使用基于函数调用的 SQL 接口定义和修改审计过滤器。要显示审计过滤器，请查询`mysql.audit_log_filter`表。

+   审计过滤器定义存储在`mysql`系统数据库的表中。

+   在给定会话中，只读的`audit_log_filter_id`系统变量的值指示是否为会话分配了过滤器。

注意

默认情况下，基于规则的审计日志过滤不记录任何用户的可审计事件。要记录所有用户的所有可审计事件，请使用以下语句，创建一个简单的过滤器以启用日志记录并将其分配给默认账户：

```sql
SELECT audit_log_filter_set_filter('log_all', '{ "filter": { "log": true } }');
SELECT audit_log_filter_set_user('%', 'log_all');
```

`%`分配的过滤器用于来自未明确分配过滤器的任何账户的连接（最初对所有账户都是真的）。

如前所述，审计过滤控制的 SQL 接口是基于函数的。以下列表简要总结了这些功能：

+   `audit_log_filter_set_filter()`: 定义过滤器。

+   `audit_log_filter_remove_filter()`: 移除过滤器。

+   `audit_log_filter_set_user()`: 开始过滤用户账户。

+   `audit_log_filter_remove_user()`: 停止过滤用户账户。

+   `audit_log_filter_flush()`: 刷新手动更改的过滤器表以影响正在进行的过滤。

有关使用示例和有关过滤功能的完整详细信息，请参见 Using Audit Log Filtering Functions 和 Audit Log Functions。

##### 审计日志过滤功能的约束

审计日志过滤功能受到以下约束：

+   要使用任何过滤功能，必须启用`audit_log`插件，否则会出错。此外，审计表必须存在，否则会出错。要安装`audit_log`插件及其附带的功能和表，请参见 Section 8.4.5.2, “Installing or Uninstalling MySQL Enterprise Audit”。

+   要使用任何过滤功能，用户必须具有`AUDIT_ADMIN` `SUPER`权限，否则会出错。要向用户账户授予其中一个权限，请使用此语句：

    ```sql
    GRANT *privilege* ON *.* TO *user*;
    ```

    或者，如果您希望避免授予`AUDIT_ADMIN`或`SUPER`权限，同时仍允许用户访问特定的过滤功能，“包装”存储程序可以被定义。这种技术在使用通用密钥环函数中描述了在密钥环函数的上下文中，可以用于过滤函数。

+   如果安装了`audit_log`插件但未创建相应的审计表和函数，则该插件将以传统模式运行。插件在服务器启动时将这些消息写入错误日志：

    ```sql
    [Warning] Plugin audit_log reported: 'Failed to open the audit log filter tables.'
    [Warning] Plugin audit_log reported: 'Audit Log plugin supports a filtering,
    which has not been installed yet. Audit Log plugin will run in the legacy
    mode, which will be disabled in the next release.'
    ```

    在 MySQL 8.0.34 之后不再支持的传统模式中，过滤只能基于事件账户或状态进行。详情请参阅第 8.4.5.10 节，“传统模式审计日志过滤”。

+   理论上，具有足够权限的用户可能会错误地在审计日志过滤器中创建一个“中止”项目，阻止自己和其他管理员访问系统。从 MySQL 8.0.28 开始，`AUDIT_ABORT_EXEMPT`权限可用于允许用户账户的查询始终执行，即使“中止”项目会阻止它们。因此，具有此权限的账户可以用于在审计配置错误后恢复对系统的访问。查询仍然记录在审计日志中，但由于权限的存在，不会被拒绝，而是被允许执行。

    在 MySQL 8.0.28 或更高版本中创建的具有`SYSTEM_USER`权限的账户在创建时会自动分配`AUDIT_ABORT_EXEMPT`权限。在进行 MySQL 8.0.28 或更高版本的升级过程时，如果没有现有账户被分配该权限，则具有`SYSTEM_USER`权限的现有账户也会被分配`AUDIT_ABORT_EXEMPT`权限。

##### 使用审计日志过滤功能

在使用审计日志函数之前，请根据第 8.4.5.2 节，“安装或卸载 MySQL 企业审计”中提供的说明安装它们。使用任何这些函数都需要`AUDIT_ADMIN`或`SUPER`权限。

审计日志过滤功能通过提供一个接口来创建、修改和删除过滤定义并将过滤器分配给用户账户，实现了过滤控制。

过滤器定义是`JSON`值。有关在 MySQL 中使用 `JSON` 数据的信息，请参见第 13.5 节，“JSON 数据类型”。本节显示了一些简单的过滤器定义。有关过滤器定义的更多信息，请参见第 8.4.5.8 节，“编写审计日志过滤器定义”。

当连接到达时，审计日志插件通过在当前过滤器分配中搜索用户账户名来确定新会话使用哪个过滤器：

+   如果用户被分配了过滤器，则审计日志将使用该过滤器。

+   否则，如果不存在特定于用户的过滤器分配，但是为默认账户（`%`）分配了过滤器，则审计日志将使用默认过滤器。

+   否则，审计日志不会从会话中选择任何审计事件进行处理。

如果会话期间发生更改用户操作（参见 mysql_change_user()），会话的过滤器分配将根据相同规则更新，但针对新用户。

默认情况下，没有账户被分配过滤器，因此不会对任何账户进行可审计事件的处理。

假设您想要更改默认设置为仅记录与连接相关的活动（例如，查看连接、更改用户和断开连接事件，但不记录用户在连接时执行的 SQL 语句）。为实现此目的，请定义一个过滤器（此处命名为 `log_conn_events`），仅允许记录 `connection` 类中的事件，并将该过滤器分配给默认账户，表示为 `%` 账户名：

```sql
SET @f = '{ "filter": { "class": { "name": "connection" } } }';
SELECT audit_log_filter_set_filter('log_conn_events', @f);
SELECT audit_log_filter_set_user('%', 'log_conn_events');
```

现在审计日志将对没有明确定义过滤器的任何账户的连接使用此默认账户过滤器。

要明确为特定用户账户或多个账户分配过滤器，请定义过滤器，然后将其分配给相关账户：

```sql
SELECT audit_log_filter_set_filter('log_all', '{ "filter": { "log": true } }');
SELECT audit_log_filter_set_user('user1@localhost', 'log_all');
SELECT audit_log_filter_set_user('user2@localhost', 'log_all');
```

现在 `user1@localhost` 和 `user2@localhost` 启用了完整记录。其他账户的连接将继续使用默认账户过滤器进行过滤。

要取消用户账户与当前过滤器的关联，要么取消分配过滤器，要么分配不同的过滤器：

+   要取消用户账户的过滤器分配：

    ```sql
    SELECT audit_log_filter_remove_user('user1@localhost');
    ```

    账户当前会话的过滤保持不受影响。如果账户有默认账户过滤器，则后续连接将使用该过滤器进行过滤，否则不记录。

+   要为用户账户分配不同的过滤器：

    ```sql
    SELECT audit_log_filter_set_filter('log_nothing', '{ "filter": { "log": false } }');
    SELECT audit_log_filter_set_user('user1@localhost', 'log_nothing');
    ```

    账户当前会话的过滤保持不受影响。后续连接将使用新过滤器进行过滤。对于此处显示的过滤器，这意味着来自 `user1@localhost` 的新连接不会记录。

对于审计日志过滤，用户名和主机名的比较是区分大小写的。这与用于权限检查的主机名比较不区分大小写不同。

要移除过滤器，请执行以下操作：

```sql
SELECT audit_log_filter_remove_filter('log_nothing');
```

移除过滤器还会取消分配给它的任何用户，包括这些用户的当前会话。

刚刚描述的过滤函数立即影响审计过滤，并更新存储过滤器和用户帐户的 `mysql` 系统数据库中的审计日志表（参见 Audit Log Tables）。也可以直接修改审计日志表，使用诸如 `INSERT`、`UPDATE` 和 `DELETE` 等语句，但这些更改不会立即影响过滤。要刷新更改并使其生效，请调用 `audit_log_filter_flush()`：

```sql
SELECT audit_log_filter_flush();
```

警告

`audit_log_filter_flush()` 应仅在直接修改审计表后使用，以强制重新加载所有过滤器。否则，应避免使用此函数。实际上，这是使用 `UNINSTALL PLUGIN` 和 `INSTALL PLUGIN` 卸载和重新加载 `audit_log` 插件的简化版本。

`audit_log_filter_flush()` 影响所有当前会话，并将它们从先前的过滤器中分离。当前会话将不再记录，除非它们断开连接并重新连接，或执行更改用户操作。

要确定过滤器是否分配给当前会话，请检查只读的 `audit_log_filter_id` 系统变量的会话值。如果值为 0，则未分配过滤器。非零值表示分配的过滤器的内部维护 ID：

```sql
mysql> SELECT @@audit_log_filter_id;
+-----------------------+
| @@audit_log_filter_id |
+-----------------------+
|                     2 |
+-----------------------+
```
