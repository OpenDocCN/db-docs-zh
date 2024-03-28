# 28.4.8 The INFORMATION_SCHEMA INNODB_CMP_PER_INDEX and INNODB_CMP_PER_INDEX_RESET Tables

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-cmp-per-index-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-cmp-per-index-table.html)

`INNODB_CMP_PER_INDEX` 和 `INNODB_CMP_PER_INDEX_RESET` 表提供与压缩 `InnoDB`表和索引相关的操作的状态信息，针对每个数据库、表和索引的组合分别提供统计信息，以帮助您评估特定表的压缩性能和实用性。

对于压缩的`InnoDB`表，表数据和所有二级索引都被压缩。在这种情况下，表数据被视为另一个索引，恰好包含所有列：聚簇索引。

`INNODB_CMP_PER_INDEX` 和 `INNODB_CMP_PER_INDEX_RESET` 表具有以下列：

+   `DATABASE_NAME`

    包含适用表的模式（数据库）。

+   `TABLE_NAME`

    监控压缩统计信息的表。

+   `INDEX_NAME`

    用于监控压缩统计信息的索引。

+   `COMPRESS_OPS`

    尝试的压缩操作次数。每当创建一个空页面或未压缩修改日志的空间用尽时，就会对页面进行压缩。

+   `COMPRESS_OPS_OK`

    成功的压缩操作次数。从`COMPRESS_OPS`值中减去以获取压缩失败的次数。除以`COMPRESS_OPS`值以获取压缩失败的百分比。

+   `COMPRESS_TIME`

    用于在此索引���压缩数据的总时间（以秒为单位）。

+   `UNCOMPRESS_OPS`

    执行的解压操作次数。每当压缩失败或在缓冲池中第一次访问压缩页面且未压缩页面不存在时，压缩的`InnoDB`页面就会被解压。

+   `UNCOMPRESS_TIME`

    用于在此索引中解压数据的总时间（以秒为单位）。

#### 示例

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_CMP_PER_INDEX\G
*************************** 1\. row ***************************
  database_name: employees
     table_name: salaries
     index_name: PRIMARY
   compress_ops: 0
compress_ops_ok: 0
  compress_time: 0
 uncompress_ops: 23451
uncompress_time: 4
*************************** 2\. row ***************************
  database_name: employees
     table_name: salaries
     index_name: emp_no
   compress_ops: 0
compress_ops_ok: 0
  compress_time: 0
 uncompress_ops: 1597
uncompress_time: 0
```

#### 注意事项

+   使用这些表来衡量特定表、索引或两者的 `InnoDB` 表 压缩 效果。

+   您必须具有 `PROCESS` 权限才能查询这些表。

+   使用 `INFORMATION_SCHEMA` `COLUMNS` 表或 `SHOW COLUMNS` 语句查看有关这些表的列的其他信息，包括数据类型和默认值。

+   由于为每个索引收集单独的测量数据会带来很大的性能开销，`INNODB_CMP_PER_INDEX` 和 `INNODB_CMP_PER_INDEX_RESET` 统计数据默认不会被收集。在执行想要监视的压缩表操作之前，您必须启用 `innodb_cmp_per_index_enabled` 系统变量。

+   有关使用信息，请参阅 Section 17.9.1.4, “Monitoring InnoDB Table Compression at Runtime” 和 Section 17.15.1.3, “Using the Compression Information Schema Tables”。有关 `InnoDB` 表压缩的一般信息，请参阅 Section 17.9, “InnoDB Table and Page Compression”。
