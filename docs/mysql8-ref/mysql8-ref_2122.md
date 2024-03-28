# 29.18 性能模式和插件

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-and-plugins.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-and-plugins.html)

使用`UNINSTALL PLUGIN`命令移除插件不会影响已经收集的插件中的代码信息。即使稍后卸载插件，执行插件加载时花费的时间仍然会被计算在内。相关的事件信息，包括聚合信息，仍然可以在`performance_schema`数据库表中读取。有关插件安装和移除的更多信息，请参见第 29.7 节，“性能模式状态监控”。

实现插件代码的插件开发者应该记录其插装特性，以便加载插件的人能够考虑其需求。例如，第三方存储引擎应该在文档中说明引擎需要多少内存用于互斥锁和其他插装。
