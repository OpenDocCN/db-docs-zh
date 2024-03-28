# 29.12.2 性能模式设置表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-setup-tables.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-setup-tables.html)

29.12.2.1 设置演员表

29.12.2.2 设置消费者表

29.12.2.3 设置仪器表

29.12.2.4 设置对象表

29.12.2.5 设置线程表

设置表提供有关当前仪表化的信息，并允许更改监视配置。因此，如果您拥有`UPDATE`权限，则可以更改这些表中的某些列。

使用表而不是单个变量来存储设置信息，在修改性能模式配置方面提供了高度的灵活性。例如，您可以使用标准 SQL 语法的单个语句进行多个同时配置更改。

可用的设置表包括：

+   `setup_actors`：如何为新的前台线程初始化监视

+   `setup_consumers`：可以发送和存储事件信息的目的地

+   `setup_instruments`：可以收集事件的被检测对象的类别

+   `setup_objects`：应该监视哪些对象

+   `setup_threads`：被检测线程的名称和属性
