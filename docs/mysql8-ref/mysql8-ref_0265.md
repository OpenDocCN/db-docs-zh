> 原文：[`dev.mysql.com/doc/refman/8.0/en/error-log-filtering.html`](https://dev.mysql.com/doc/refman/8.0/en/error-log-filtering.html)

#### 7.4.2.4 错误日志过滤类型

错误日志配置通常包括一个日志过滤组件和一个或多个日志接收组件。对于错误日志过滤，MySQL 提供了多种组件选择：

+   `log_filter_internal`：该过滤组件基于日志事件优先级和错误代码提供错误日志过滤，结合`log_error_verbosity`和`log_error_suppression_list`系统变量。`log_filter_internal`是内置的，默认启用。参见 Section 7.4.2.5, “基于优先级的错误日志过滤（log_filter_internal）”。

+   `log_filter_dragnet`：该过滤组件基于用户提供的规则，结合`dragnet.log_error_filter_rules`系统变量，提供错误日志过滤。参见 Section 7.4.2.6, “基于规则的错误日志过滤（log_filter_dragnet）”。
