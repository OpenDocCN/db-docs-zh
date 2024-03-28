> 原文：[`dev.mysql.com/doc/refman/8.0/en/performance-schema-binary-log-transaction-compression-stats-table.html`](https://dev.mysql.com/doc/refman/8.0/en/performance-schema-binary-log-transaction-compression-stats-table.html)

#### 29.12.11.16 `binary_log_transaction_compression_stats`表

此表显示写入二进制日志和中继日志的交易有效负载的统计信息，并可用于计算启用二进制日志事务压缩的效果。有关二进制日志事务压缩的信息，请参见 Section 7.4.4.5, “Binary Log Transaction Compression”。

当服务器实例具有二进制日志，并且系统变量`binlog_transaction_compression`设置为`ON`时，才会填充`binary_log_transaction_compression_stats`表。统计数据涵盖了从服务器启动或表被截断时开始写入二进制日志和中继日志的所有交易。压缩的交易按使用的压缩算法分组，未压缩的交易与压缩算法为`NONE`一起分组，因此可以计算压缩比率。

`binary_log_transaction_compression_stats`表具有以下列：

+   `LOG_TYPE`

    这些交易是否被写入了二进制日志或中继日志。

+   `COMPRESSION_TYPE`

    用于压缩交易有效负载的压缩算法。`NONE`表示这些交易的有效负载未经压缩，在许多情况下是正确的（参见 Section 7.4.4.5, “Binary Log Transaction Compression”）。

+   `TRANSACTION_COUNTER`

    使用此压缩类型写入此日志类型的交易数量。

+   `COMPRESSED_BYTES`

    经过压缩并写入此日志类型的总字节数，计算压缩后的字节数。

+   `UNCOMPRESSED_BYTES`

    此日志类型和此压缩类型在压缩前的总字节数。

+   `COMPRESSION_PERCENTAGE`

    此日志类型和此压缩类型的压缩比率，以百分比表示。

+   `FIRST_TRANSACTION_ID`

    使用此压缩类型写入此日志类型的第一笔交易的 ID。

+   `FIRST_TRANSACTION_COMPRESSED_BYTES`

    经过压缩并写入第一笔交易的日志的总字节数，计算压缩后的字节数。

+   `FIRST_TRANSACTION_UNCOMPRESSED_BYTES`

    第一笔交易在压缩前的总字节数。

+   `FIRST_TRANSACTION_TIMESTAMP`

    第一笔交易写入日志时的时间戳。

+   `LAST_TRANSACTION_ID`

    使用此压缩类型写入此日志类型的最近一笔交易的 ID。

+   `LAST_TRANSACTION_COMPRESSED_BYTES`

    最近一笔交易在压缩后写入日志的总字节数，压缩后计算。

+   `LAST_TRANSACTION_UNCOMPRESSED_BYTES`

    最近一笔交易在压缩前的总字节数。

+   `LAST_TRANSACTION_TIMESTAMP`

    最近一笔交易被写入日志的时间戳。

`binary_log_transaction_compression_stats` 表没有索引。

`TRUNCATE TABLE` 允许用于 `binary_log_transaction_compression_stats` 表。
