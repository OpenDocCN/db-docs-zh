# 29.12.14 性能模式系统变量表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-system-variable-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-system-variable-tables.html)

29.12.14.1 性能模式 persisted_variables 表

29.12.14.2 性能模式 variables_info 表

MySQL 服务器维护许多指示其配置方式的系统变量（参见 Section 7.1.8, “服务器系统变量”）。系统变量信息可以在这些性能模式表中找到：

+   `global_variables`: 全局系统变量。只需要全局值的应用程序应该使用这个表。

+   `session_variables`: 当前会话的系统变量。想要获取自己会话的所有系统变量值的应用程序应该使用这个表。它包括会话的会话变量，以及没有会话对应的全局变量的值。

+   `variables_by_thread`: 每个活动会话的会话系统变量。想要了解特定会话的会话变量值的应用程序应该使用这个表。它只包括会话变量，通过线程 ID 标识。

+   `persisted_variables`: 提供一个 SQL 接口，用于存储持久化全局系统变量设置的`mysqld-auto.cnf`文件。参见 Section 29.12.14.1, “性能模式 persisted_variables 表”。

+   `variables_info`: 显示每个系统变量最近设置的来源以及其值范围。参见 Section 29.12.14.2, “性能模式 variables_info 表”。

查看这些表中敏感系统变量的值需要`SENSITIVE_VARIABLES_OBSERVER`权限。

会话变量表（`session_variables`、`variables_by_thread`）仅包含活动会话的信息，而不包括已终止会话。

`global_variables` 和 `session_variables` 表具有以下列：

+   `VARIABLE_NAME`

    系统变量名称。

+   `VARIABLE_VALUE`

    系统变量值。对于 `global_variables`，此列包含全局值。对于 `session_variables`，此列包含当前会话中生效的变量值。

`global_variables` 和 `session_variables` 表具有以下索引：

+   主键为 (`VARIABLE_NAME`)

`variables_by_thread` 表具有以下列：

+   `THREAD_ID`

    定义系统变量的会话的线程标识符。

+   `VARIABLE_NAME`

    系统变量名称。

+   `VARIABLE_VALUE`

    由 `THREAD_ID` 列命名的会话的会话变量值。

`variables_by_thread` 表具有以下索引：

+   主键为 (`THREAD_ID`, `VARIABLE_NAME`)

`variables_by_thread` 表仅包含关于前台线程的系统变量信息。如果性能模式未对所有线程进行仪器化，则此表会缺少一些行。在这种情况下，`Performance_schema_thread_instances_lost` 状态变量大于零。

不支持对性能模式系统变量表使用 `TRUNCATE TABLE`。
