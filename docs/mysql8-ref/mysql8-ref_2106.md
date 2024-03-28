# 29.12.21 性能模式杂项表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-miscellaneous-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-miscellaneous-tables.html)

29.12.21.1 component_scheduler_tasks 表

29.12.21.2 error_log 表

29.12.21.3 host_cache 表

29.12.21.4 innodb_redo_log_files 表

29.12.21.5 log_status 表

29.12.21.6 performance_timers 表

29.12.21.7 processlist 表

29.12.21.8 threads 表

29.12.21.9 tls_channel_status 表

29.12.21.10 user_defined_functions 表

以下各节描述了不属于前述各节讨论的表类别的表：

+   `component_scheduler_tasks`: 每个计划任务的当前状态。

+   `error_log`: 写入错误日志的最新事件。

+   `host_cache`: 内部主机缓存的信息。

+   `innodb_redo_log_files`: InnoDB 重做日志文件的信息。

+   `log_status`: 用于备份目的的服务器日志信息。

+   `performance_timers`: 可用的事件计时器。

+   `processlist`: 有关服务器进程的信息。

+   `threads`: 有关服务器线程的信息。

+   `tls_channel_status`: 连接接口的 TLS 上下文属性。

+   `user_defined_functions`: 组件、插件或`CREATE FUNCTION`语句注册的可加载函数。
