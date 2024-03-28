> 原文：[`dev.mysql.com/doc/refman/8.0/en/audit-log-legacy-filtering.html`](https://dev.mysql.com/doc/refman/8.0/en/audit-log-legacy-filtering.html)

#### 8.4.5.10 遗留模式审计日志过滤

注意

此部分描述了遗留审计日志过滤，适用于安装了`audit_log`插件但没有规则过滤所需的审计表和函数的情况。

遗留模式审计日志过滤已在 MySQL 8.0.34 中弃用。

审计日志插件可以过滤审计事件。这使您可以控制基于事件来源账户或事件状态将审计事件写入审计日志文件。连接事件和语句事件的状态过滤是分开进行的。

+   按账户进行遗留事件过滤

+   按状态进行遗留事件过滤

##### 按账户进行遗留事件过滤

要根据发起账户过滤审计事件，请在服务器启动或运行时设置以下系统变量中的一个（而不是两个）。这些已弃用的变量仅适用于遗留审计日志过滤。

+   `audit_log_include_accounts`：要包含在审计日志中的账户。如果设置了此变量，则只对这些账户进行审计。

+   `audit_log_exclude_accounts`：要排除在审计日志中的账户。如果设置了此变量，则除了这些账户外，所有其他账户都会被审计。

任一变量的值可以是`NULL`，或包含一个或多个以`*`user_name`*@*`host_name`*`格式逗号分隔的账户名字符串。默认情况下，这两个变量都是`NULL`，在这种情况下，不进行账户过滤，对所有账户进行审计。

对`audit_log_include_accounts`或`audit_log_exclude_accounts`的修改仅影响在修改后创建的连接，而不影响现有连接。

示例：要仅为`user1`和`user2`本地主机账户启用审计日志记录，请像这样设置`audit_log_include_accounts`系统变量：

```sql
SET GLOBAL audit_log_include_accounts = 'user1@localhost,user2@localhost';
```

`audit_log_include_accounts`或`audit_log_exclude_accounts`只能同时有一个为非`NULL`：

+   如果设置了`audit_log_include_accounts`，服务器会将`audit_log_exclude_accounts`设置为`NULL`。

+   如果您尝试设置`audit_log_exclude_accounts`，除非`audit_log_include_accounts`为`NULL`，否则会出现错误。在这种情况下，您必须首先通过将其设置为`NULL`来清除`audit_log_include_accounts`。

```sql
-- This sets audit_log_exclude_accounts to NULL
SET GLOBAL audit_log_include_accounts = *value*;

-- This fails because audit_log_include_accounts is not NULL
SET GLOBAL audit_log_exclude_accounts = *value*;

-- To set audit_log_exclude_accounts, first set
-- audit_log_include_accounts to NULL
SET GLOBAL audit_log_include_accounts = NULL;
SET GLOBAL audit_log_exclude_accounts = *value*;
```

如果检查任一变量的值，请注意`SHOW VARIABLES`将`NULL`显示为空字符串。要将`NULL`显示为`NULL`，请改用`SELECT`：

```sql
mysql> SHOW VARIABLES LIKE 'audit_log_include_accounts';
+----------------------------+-------+
| Variable_name              | Value |
+----------------------------+-------+
| audit_log_include_accounts |       |
+----------------------------+-------+
mysql> SELECT @@audit_log_include_accounts;
+------------------------------+
| @@audit_log_include_accounts |
+------------------------------+
| NULL                         |
+------------------------------+
```

如果用户名或主机名需要引用，因为它包含逗号、空格或其他特殊字符，请使用单引号引用。如果变量值本身用单引号引用，则每个内部单引号都要加倍或用反斜杠转义。以下语句分别为本地`root`账户启用审计日志记录，并且虽然引用样式不同，但是等效的：

```sql
SET GLOBAL audit_log_include_accounts = 'root@localhost';
SET GLOBAL audit_log_include_accounts = '''root''@''localhost''';
SET GLOBAL audit_log_include_accounts = '\'root\'@\'localhost\'';
SET GLOBAL audit_log_include_accounts = "'root'@'localhost'";
```

如果启用了`ANSI_QUOTES` SQL 模式，则最后一个语句将无效，因为在该模式下，双引号表示标识符引用，而不是字符串引用。

##### 按状态进行传统事件过滤

要根据状态过滤审计事件，请在服务器启动或运行时设置以下系统变量。这些已弃用的变量仅适用于传统审计日志过滤。对于 JSON 审计日志过滤，不同的状态变量适用；请参阅 Audit Log Options and Variables。

+   `audit_log_connection_policy`：连接事件的记录策略

+   `audit_log_statement_policy`：语句事件的记录策略

每个变量都可以取值`ALL`（记录所有相关事件；这是默认值）、`ERRORS`（仅记录失败事件）或`NONE`（不记录事件）。例如，要记录所有语句事件但仅记录失败的连接事件，请使用以下设置：

```sql
SET GLOBAL audit_log_statement_policy = ALL;
SET GLOBAL audit_log_connection_policy = ERRORS;
```

另一个策略系统变量`audit_log_policy`可用，但不像`audit_log_connection_policy`和`audit_log_statement_policy`那样提供更多控制。它只能在服务器启动时设置。

注意

自 MySQL 8.0.34 起，`audit_log_policy`传统模式系统变量已弃用。

在运行时，它是一个只读变量。它可以取`ALL`（记录所有事件；这是默认值）、`LOGINS`（记录连接事件）、`QUERIES`（记录语句事件）或`NONE`（不记录事件）的值。对于这些值中的任何一个，审计日志插件都会记录所有选定的事件，不区分成功或失败。在启动时使用`audit_log_policy`的工作方式如下：

+   如果您未设置`audit_log_policy`或将其设置为默认值`ALL`，则对于`audit_log_connection_policy`或`audit_log_statement_policy`的任何显式设置均按照指定的方式应用。如果未指定，则它们默认为`ALL`。

+   如果您将`audit_log_policy`设置为非`ALL`值，则该值优先，并用于设置`audit_log_connection_policy`和`audit_log_statement_policy`，如下表所示。如果您还将这些变量中的任一设置为非默认值`ALL`，服务器将向错误日志写入消息，指示它们的值被覆盖。

    | 启动时的 audit_log_policy 值 | 结果的 audit_log_connection_policy 值 | 结果的 audit_log_statement_policy 值 |
    | --- | --- | --- |
    | `LOGINS` | `ALL` | `NONE` |
    | `QUERIES` | `NONE` | `ALL` |
    | `NONE` | `NONE` | `NONE` |
