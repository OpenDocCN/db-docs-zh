> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-sys-config.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-sys-config.html)

#### 30.4.2.1 `sys_config`表

此表包含`sys`模式配置选项，每个选项一行。通过更新此表进行的配置更改会跨客户端会话和服务器重启持久保存。

`sys_config`表具有以下列：

+   `variable`

    配置选项名称。

+   `value`

    配置选项值。

+   `set_time`

    最近修改行的时间戳。

+   `set_by`

    最近修改行的帐户。如果自`sys`模式安装以来未更改行，则值为`NULL`。

为了最小化从`sys_config`表直接读取的次数，`sys`模式函数会检查具有相应名称的用户定义变量，该变量是具有相同名称加上`@sys.`前缀的用户定义变量。（例如，与`diagnostics.include_raw`选项对应的变量是`@sys.diagnostics.include_raw`。）如果用户定义变量存在于当前会话中且非`NULL`，函数会优先使用其值，而不是`sys_config`表中的值。否则，函数会读取并使用表中的值。在后一种情况下，调用函数通常还会将相应的用户定义变量设置为表中的值，以便同一会话中对配置选项的进一步引用使用该变量，无需再次读取表。

例如，`statement_truncate_len`选项控制`format_statement()` Function")函数返回的语句的最大长度。默认值为 64。要临时将值更改为 32 以用于当前会话，请设置相应的`@sys.statement_truncate_len`用户定义变量：

```sql
mysql> SET @stmt = 'SELECT variable, value, set_time, set_by FROM sys_config';
mysql> SELECT sys.format_statement(@stmt);
+----------------------------------------------------------+
| sys.format_statement(@stmt)                              |
+----------------------------------------------------------+
| SELECT variable, value, set_time, set_by FROM sys_config |
+----------------------------------------------------------+
mysql> SET @sys.statement_truncate_len = 32;
mysql> SELECT sys.format_statement(@stmt);
+-----------------------------------+
| sys.format_statement(@stmt)       |
+-----------------------------------+
| SELECT variabl ... ROM sys_config |
+-----------------------------------+
```

会话中后续调用`format_statement()` Function")将继续使用用户定义变量值（32），而不是表中存储的值（64）。

要停止使用用户定义变量并恢复使用表中的值，请在会话中将变量设置为`NULL`：

```sql
mysql> SET @sys.statement_truncate_len = NULL;
mysql> SELECT sys.format_statement(@stmt);
+----------------------------------------------------------+
| sys.format_statement(@stmt)                              |
+----------------------------------------------------------+
| SELECT variable, value, set_time, set_by FROM sys_config |
+----------------------------------------------------------+
```

或者，结束当前会话（导致用户定义变量不再存在）并开始新会话。

在`sys_config`表中描述的传统关系可以被利用来进行临时配置更改，这些更改在您的会话结束时结束。然而，如果您设置了一个用户定义变量，然后在同一会话中随后更改相应的表值，只要用户定义变量存在非`NULL`值，该会话中不会使用更改后的表值。（更改后的表值会在未分配用户定义变量的其他会话中使用。）

以下列表描述了`sys_config`表中的选项及相应的用户定义变量：

+   `diagnostics.allow_i_s_tables`，`@sys.diagnostics.allow_i_s_tables`

    如果此选项为`ON`，则允许`diagnostics()`过程")过程在 Information Schema `TABLES`表上执行表扫描。如果有许多表，这可能很昂贵。默认值为`OFF`。

+   `diagnostics.include_raw`，`@sys.diagnostics.include_raw`

    如果此选项为`ON`，则`diagnostics()`过程")过程将包括从查询`metrics`视图获取的原始输出。默认值为`OFF`。

+   `ps_thread_trx_info.max_length`，`@sys.ps_thread_trx_info.max_length`

    由`ps_thread_trx_info()`函数")函数生成的 JSON 输出的最大长度。默认值为 65535。

+   `statement_performance_analyzer.limit`，`@sys.statement_performance_analyzer.limit`

    为没有内置限制的视图返回的最大行数。（例如，`statements_with_runtimes_in_95th_percentile`视图在某种意义上具有内置限制，它仅返回在第 95 百分位数中具有平均执行时间的语句。）默认值为 100。

+   `statement_performance_analyzer.view`，`@sys.statement_performance_analyzer.view`

    `statement_performance_analyzer()` Procedure") 过程要使用的自定义查询或视图（它本身被 `diagnostics()` Procedure") 过程调用）。如果选项值包含空格，则将其解释为查询。否则，它必须是查询 Performance Schema `events_statements_summary_by_digest` 表的现有视图的名称。如果 `statement_performance_analyzer.limit` 配置选项大于 0，则查询或视图定义中不能有 `LIMIT` 子句。默认值为 `NULL`（未定义自定义视图）。

+   `statement_truncate_len`, `@sys.statement_truncate_len`

    `format_statement()` Function") 函数返回的语句的最大长度。超过此长度的语句将被截断。默认值为 64。

其他选项可以添加到 `sys_config` 表中。例如，`diagnostics()` Procedure") 和 `execute_prepared_stmt()` Procedure") 过程如果存在 `debug` 选项，则使用该选项，但默认情况下，此选项不是 `sys_config` 表的一部分，因为调试输出通常只是临时启用，通过设置相应的 `@sys.debug` 用户定义变量。要在不必在各个会话中设置该变量的情况下启用调试输出，将选项添加到表中：

```sql
mysql> INSERT INTO sys.sys_config (variable, value) VALUES('debug', 'ON');
```

要更改表中的调试设置，需要做两件事。首先，修改表中的值：

```sql
mysql> UPDATE sys.sys_config
       SET value = 'OFF'
       WHERE variable = 'debug';
```

其次，为了确保当前会话中的过程调用使用表中的更改值，将相应的用户定义变量设置为 `NULL`：

```sql
mysql> SET @sys.debug = NULL;
```
