# 28.4.6 INFORMATION_SCHEMA INNODB_CMP 和 INNODB_CMP_RESET 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-cmp-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-cmp-table.html)

`INNODB_CMP`和`INNODB_CMP_RESET`表提供与压缩`InnoDB`表相关操作的状态信息。

`INNODB_CMP`和`INNODB_CMP_RESET`表具有以下列：

+   `PAGE_SIZE`

    压缩页面的大小（以字节为单位）。

+   `COMPRESS_OPS`

    一个大小为`PAGE_SIZE`的 B 树页面被压缩的次数。每当创建一个空页面或未压缩修改日志的空间用尽时，页面就会被压缩。

+   `COMPRESS_OPS_OK`

    一个大小为`PAGE_SIZE`的 B 树页面成功压缩的次数。此计数永远不应超过`COMPRESS_OPS`。

+   `COMPRESS_TIME`

    用于尝试压缩大小为`PAGE_SIZE`的 B 树页面所用的总时间（以秒为单位）。

+   `UNCOMPRESS_OPS`

    一个大小为`PAGE_SIZE`的 B 树页面被解压的次数。每当压缩失败或在缓冲池中不存在未压缩页面时首次访问���，B 树页面就会被解压。

+   `UNCOMPRESS_TIME`

    用于解压大小为`PAGE_SIZE`的 B 树页面所用的总时间（以秒为单位）。

#### 示例

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_CMP\G
*************************** 1\. row ***************************
      page_size: 1024
   compress_ops: 0
compress_ops_ok: 0
  compress_time: 0
 uncompress_ops: 0
uncompress_time: 0
*************************** 2\. row ***************************
      page_size: 2048
   compress_ops: 0
compress_ops_ok: 0
  compress_time: 0
 uncompress_ops: 0
uncompress_time: 0
*************************** 3\. row ***************************
      page_size: 4096
   compress_ops: 0
compress_ops_ok: 0
  compress_time: 0
 uncompress_ops: 0
uncompress_time: 0
*************************** 4\. row ***************************
      page_size: 8192
   compress_ops: 86955
compress_ops_ok: 81182
  compress_time: 27
 uncompress_ops: 26828
uncompress_time: 5
*************************** 5\. row ***************************
      page_size: 16384
   compress_ops: 0
compress_ops_ok: 0
  compress_time: 0
 uncompress_ops: 0
uncompress_time: 0
```

#### 注意

+   使用这些表来衡量数据库中`InnoDB`表压缩的有效性。

+   您必须具有`PROCESS`权限才能查询此表。

+   使用`INFORMATION_SCHEMA``COLUMNS`表或`SHOW COLUMNS`语句查看有关此表的列的其他信息，包括数据类型和默认值。

+   有关使用信息，请参见第 17.9.1.4 节，“在运行时监视 InnoDB 表压缩”和第 17.15.1.3 节，“使用压缩信息模式表”。有关`InnoDB`表压缩的一般信息，请参见第 17.9 节，“InnoDB 表和页面压缩”。
