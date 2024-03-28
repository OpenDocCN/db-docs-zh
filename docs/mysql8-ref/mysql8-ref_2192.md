> 原文：[`dev.mysql.com/doc/refman/8.0/en/sys-diagnostics.html`](https://dev.mysql.com/doc/refman/8.0/en/sys-diagnostics.html)

#### 30.4.4.2 The diagnostics() Procedure

为诊断目的创建当前服务器状态报告。

此过程在执行期间通过操纵`sql_log_bin`系统变量的会话值来禁用二进制日志记录。这是一项受限制的操作，因此该过程需要具有足够权限设置受限制会话变量的权限。请参阅第 7.1.9.1 节，“系统变量权限”。

为`diagnostics()` Procedure")收集的数据包括以下信息：

+   来自`metrics`视图的信息（参见第 30.4.3.21 节，“The metrics View”）

+   来自其他相关`sys`模式视图的信息，例如确定第 95 百分位数中的查询的视图

+   如果 MySQL 服务器是 NDB Cluster 的一部分，则包括来自`ndbinfo`模式的信息

+   复制状态（源和副本）

一些 sys 模式视图被计算为初始（可选）、总体和增量值：

+   初始视图是`diagnostics()` Procedure")过程开始时视图的内容。此输出与用于增量视图的开始值相同。如果`diagnostics.include_raw`配置选项为`ON`，则包括初始视图。

+   总体视图是`diagnostics()` Procedure")过程结束时视图的内容。此输出与用于增量视图的结束值相同。总体视图始终包括在内。

+   增量视图是过程执行开始到结束的差异。最小值和最大值分别是结束视图中的最小值和最大值。它们不一定反映监控期间的最小值和最大值。除了`metrics`视图外，增量仅在第一个和最后一个输出之间计算。

##### 参数

+   `in_max_runtime INT UNSIGNED`: 数据收集的最大时间，单位为秒。使用`NULL`以收集默认的 60 秒的数据。否则，使用大于 0 的值。

+   `in_interval INT UNSIGNED`: 数据收集之间的睡眠时间，单位为秒。使用`NULL`以睡眠默认的 30 秒。否则，使用大于 0 的值。

+   `in_auto_config ENUM('current', 'medium', 'full')`: 要使用的性能模式配置。允许的值为：

    +   `current`: 使用当前仪器和消费者设置。

    +   `medium`: 启用一些仪器和消费者。

    +   `full`：启用所有仪器和消费者。

    注意

    启用的仪器和消费者越多，对 MySQL 服务器性能的影响就越大。要小心使用`medium`设置，尤其是`full`设置，对性能影响很大。

    使用`medium`或`full`设置需要`SUPER`特权。

    如果选择的设置不是`current`，则在过程结束时将恢复当前设置。

##### 配置选项

`diagnostics()` Procedure")操作可以使用以下配置选项或其对应的用户定义变量进行修改（参见 Section 30.4.2.1, “The sys_config Table”）：

+   `debug`，`@sys.debug`

    如果此选项为`ON`，则生成调试输出。默认为`OFF`。

+   `diagnostics.allow_i_s_tables`，`@sys.diagnostics.allow_i_s_tables`

    如果此选项为`ON`，则允许`diagnostics()` Procedure")过程在 Information Schema `TABLES`表上执行表扫描。如果表很多，这可能会很昂贵。默认为`OFF`。

+   `diagnostics.include_raw`，`@sys.diagnostics.include_raw`

    如果此选项为`ON`，则`diagnostics()` Procedure")过程的输出包括查询`metrics`视图的原始输出。默认为`OFF`。

+   `statement_truncate_len`，`@sys.statement_truncate_len`

    `format_statement()` Function")函数返回的语句的最大长度。超过此长度的语句将被截断。默认为 64。

##### 示例

创建一个诊断报告，每 30 秒开始一次迭代，最多运行 120 秒，使用当前的 Performance Schema 设置：

```sql
mysql> CALL sys.diagnostics(120, 30, 'current');
```

要在运行时将`diagnostics()`过程的输出捕获到文件中，请使用**mysql**客户端的`tee `filename``和`notee`命令（参见 Section 6.5.1.2, “mysql Client Commands”）：

```sql
mysql> tee diag.out;
mysql> CALL sys.diagnostics(120, 30, 'current');
mysql> notee;
```
