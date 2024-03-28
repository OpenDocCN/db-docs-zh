# 29.2 性能模式构建配置

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-build-configuration.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-build-configuration.html)

性能模式是强制性的，并且始终在编译中。可以排除性能模式仪表的某些部分。例如，要排除阶段和语句仪表，可以执行以下操作：

```sql
$> cmake . \
        -DDISABLE_PSI_STAGE=1 \
        -DDISABLE_PSI_STATEMENT=1
```

更多信息，请参阅第 2.8.7 节，“MySQL 源配置选项”中`DISABLE_PSI_*`XXX`*` **CMake** 选项的描述。

如果您在之前没有配置性能模式（或者使用缺失或过时表的旧版本性能模式）的情况下安装 MySQL，那么在安装 MySQL 时可能会出现以下错误日志中的消息：

```sql
[ERROR] Native table 'performance_schema'.'events_waits_history'
has the wrong structure
[ERROR] Native table 'performance_schema'.'events_waits_history_long'
has the wrong structure
...
```

要解决这个问题，请执行 MySQL 升级过程。参见第三章，*升级 MySQL*。

因为性能模式在构建时配置到服务器中，所以在`SHOW ENGINES`的输出中会出现`PERFORMANCE_SCHEMA`的行。这意味着性能模式是可用的，而不是已启用。要启用它，必须在服务器启动时执行，如下一节所述。
