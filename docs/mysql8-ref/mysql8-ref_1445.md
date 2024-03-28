> 原文：[`dev.mysql.com/doc/refman/8.0/en/replication-features-json.html`](https://dev.mysql.com/doc/refman/8.0/en/replication-features-json.html)

#### 19.5.1.17 JSON 文档的复制

在 MySQL 8.0 之前，对 JSON 列的更新始终被写入二进制日志作为完整文档。在 MySQL 8.0 中，可以记录 JSON 文档的部分更新（请参阅 JSON 值的部分更新），这更有效率。记录行为取决于所使用的格式，如下所述：

**基于语句的复制。** JSON 部分更新始终被记录为部分更新。在使用基于语句的日志记录时，无法禁用此功能。

**基于行的复制。** 默认情况下，JSON 部分更新不会被记录为部分更新，而是被记录为完整文档。要启用部分更新的记录，请设置`binlog_row_value_options=PARTIAL_JSON`。如果复制源设置了此变量，则来自该源的部分更新将由副本处理和应用，而不管副本自身对该变量的设置如何。

运行 MySQL 8.0.2 或更早版本的服务器无法识别用于 JSON 部分更新的日志事件。因此，当从运行 MySQL 8.0.3 或更高版本的服务器复制到此类服务器时，必须通过将此变量设置为`''`（空字符串）在源端禁用`binlog_row_value_options`。有关更多信息，请参阅此变量的描述。
