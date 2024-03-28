# 29.4 性能模式运行时配置

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-runtime-configuration.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-runtime-configuration.html)

29.4.1 性能模式事件定时

29.4.2 性能模式事件过滤

29.4.3 事件预过滤

29.4.4 按仪器预过滤

29.4.5 按对象预过滤

29.4.6 按线程预过滤

29.4.7 按消费者预过滤

29.4.8 示例消费者配置

29.4.9 为过滤操作命名仪器或消费者

29.4.10 确定什么被仪器化

特定的性能模式功能可以在运行时启用，以控制发生哪些类型的事件收集。

性能模式设置表包含有关监控配置的信息：

```sql
mysql> SELECT TABLE_NAME FROM INFORMATION_SCHEMA.TABLES
       WHERE TABLE_SCHEMA = 'performance_schema'
       AND TABLE_NAME LIKE 'setup%';
+-------------------+
| TABLE_NAME        |
+-------------------+
| setup_actors      |
| setup_consumers   |
| setup_instruments |
| setup_objects     |
| setup_threads     |
+-------------------+
```

您可以检查这些表的内容，以获取有关性能模式监控特性的信息。如果您拥有`UPDATE`权限，可以通过修改设置表来改变性能模式的操作，从而影响监控的方式。有关这些表的更多详细信息，请参阅第 29.12.2 节，“性能模式设置表”。

`setup_instruments`和`setup_consumers`表列出了可以收集事件的仪器和实际收集事件信息的消费者类型。其他设置表可以进一步修改监控配置。第 29.4.2 节，“性能模式事件过滤”讨论了如何修改这些表以影响事件收集。

如果有必须在运行时使用 SQL 语句进行性能模式配置更改，并且希望这些更改在每次服务器启动时生效，请将这些语句放入一个文件中，并使用`init_file`系统变量设置文件名来启动服务器。如果您有多个监控配置，每个配置都针对不同类型的监控，比如常规服务器健康监控、事件调查、应用行为故障排除等，这种策略也很有用。将每个监控配置的语句放入各自的文件中，并在启动服务器时将适当的文件指定为`init_file`的值。
