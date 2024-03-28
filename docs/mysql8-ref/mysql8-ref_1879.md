# 27.4.4 事件元数据

> 原文：[`dev.mysql.com/doc/refman/8.0/en/events-metadata.html`](https://dev.mysql.com/doc/refman/8.0/en/events-metadata.html)

要获取有关事件的元数据：

+   查询`INFORMATION_SCHEMA`数据库的`EVENTS`表。参见 Section 28.3.14, “The INFORMATION_SCHEMA EVENTS Table”。

+   使用`SHOW CREATE EVENT`语句。参见 Section 15.7.7.7, “SHOW CREATE EVENT Statement”。

+   使用`SHOW EVENTS`语句。参见 Section 15.7.7.18, “SHOW EVENTS Statement”。

**事件调度器时间表示**

MySQL 中的每个会话都有一个会话时区（STZ）。这是会话`time_zone`值，当会话开始时从服务器的全局`time_zone`值初始化，但在会话期间可能会更改。

当`CREATE EVENT`或`ALTER EVENT`语句执行时，使用当前会话时区来解释事件定义中指定的时间。这成为事件时区（ETZ）；也就是用于事件调度并在事件执行时生效的时区。

为了在数据字典中表示事件信息，`execute_at`、`starts`和`ends`时间被转换为 UTC 并与事件时区一起存储。这使得事件执行可以按照定义进行，而不受服务器时区或夏令时效果的影响。`last_executed`时间也以 UTC 存储。

事件时间可以通过从信息模式`EVENTS`表或`SHOW EVENTS`中选择来获取，但它们以 ETZ 或 STZ 值报告。以下表总结了事件时间的表示。

| 值 | `EVENTS` 表 | `SHOW EVENTS` |
| --- | --- | --- |
| 执行时间 | ETZ | ETZ |
| 开始时间 | ETZ | ETZ |
| 结束时间 | ETZ | ETZ |
| 上次执行时间 | ETZ | n/a |
| 创建时间 | STZ | n/a |
| 上次修改时间 | STZ | n/a |
