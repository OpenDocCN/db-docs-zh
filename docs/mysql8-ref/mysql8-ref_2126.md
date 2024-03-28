# 29.20 关于 Performance Schema 的限制

> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-restrictions.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-restrictions.html)

Performance Schema 避免使用互斥锁来收集或生成数据，因此无法保证一致性，结果有时可能不正确。`performance_schema` 表中的事件值是不确定性的且不可重复的。

如果您将事件信息保存在另一个表中，则不应假定原始事件稍后仍然可用。例如，如果您从 `performance_schema` 表中选择事件到临时表中，打算稍后将该表与原始表连接，可能会找不到匹配项。

**mysqldump** 和 `BACKUP DATABASE` 会忽略 `performance_schema` 数据库中的表。

`performance_schema` 数据库中的表无法使用 `LOCK TABLES` 进行锁定，除了 `setup_*`xxx`*` 表。

`performance_schema` 数据库中的表无法被索引。

`performance_schema` 数据库中的表不会被复制。

计时器的类型可能因平台而异。`performance_timers` 表显示了可用的事件计时器。如果给定计时器名称在此表中的值为 `NULL`，则表示该计时器在您的平台上不受支持。

适用于存储引擎的工具可能并未为所有存储引擎实现。每个第三方引擎的工具化是引擎维护者的责任。
