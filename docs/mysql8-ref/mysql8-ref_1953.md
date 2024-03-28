# 28.4.7 INFORMATION_SCHEMA INNODB_CMPMEM 和 INNODB_CMPMEM_RESET 表

> 原文：[`dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-cmpmem-table.html`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-cmpmem-table.html)

[`INNODB_CMPMEM`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-cmpmem-table.html)和[`INNODB_CMPMEM_RESET`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-cmpmem-table.html)表提供有关`InnoDB`缓冲池内压缩[页](https://dev.mysql.com/doc/refman/8.0/en/glossary.html#glos_page)的状态信息。

[`INNODB_CMPMEM`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-cmpmem-table.html)和[`INNODB_CMPMEM_RESET`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-cmpmem-table.html)表具有以下列：

+   `PAGE_SIZE`

    字节中的块大小。此表的每个记录描述此大小的块。

+   `BUFFER_POOL_INSTANCE`

    缓冲池实例的唯一标识符。

+   `PAGES_USED`

    当前正在使用的大小为`PAGE_SIZE`的块数。

+   `PAGES_FREE`

    当前可用于分配的大小为`PAGE_SIZE`的块数。此列显示内存池中的外部碎片。理想情况下，这些数字应最多为 1。

+   `RELOCATION_OPS`

    大小为`PAGE_SIZE`的块已重新定位的次数。当尝试形成更大的释放块时，伙伴系统可以重新定位已分配的“伙伴邻居”块。从[`INNODB_CMPMEM_RESET`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-innodb-cmpmem-table.html)表中读取会重置此计数。

+   `RELOCATION_TIME`

    用于重新定位大小为`PAGE_SIZE`的块所用的总微秒数。从表`INNODB_CMPMEM_RESET`中读取会重置此计数。

#### 示例

```sql
mysql> SELECT * FROM INFORMATION_SCHEMA.INNODB_CMPMEM\G
*************************** 1\. row ***************************
           page_size: 1024
buffer_pool_instance: 0
          pages_used: 0
          pages_free: 0
      relocation_ops: 0
     relocation_time: 0
*************************** 2\. row ***************************
           page_size: 2048
buffer_pool_instance: 0
          pages_used: 0
          pages_free: 0
      relocation_ops: 0
     relocation_time: 0
*************************** 3\. row ***************************
           page_size: 4096
buffer_pool_instance: 0
          pages_used: 0
          pages_free: 0
      relocation_ops: 0
     relocation_time: 0
*************************** 4\. row ***************************
           page_size: 8192
buffer_pool_instance: 0
          pages_used: 7673
          pages_free: 15
      relocation_ops: 4638
     relocation_time: 0
*************************** 5\. row ***************************
           page_size: 16384
buffer_pool_instance: 0
          pages_used: 0
          pages_free: 0
      relocation_ops: 0
     relocation_time: 0
```

#### 注意

+   使用这些表来衡量数据库中`InnoDB`表的压缩效果。

+   您必须具有[`PROCESS`](https://dev.mysql.com/doc/refman/8.0/en/privileges-provided.html#priv_process)权限才能查询此表。

+   使用`INFORMATION_SCHEMA` [`COLUMNS`](https://dev.mysql.com/doc/refman/8.0/en/information-schema-columns-table.html)表或[`SHOW COLUMNS`](https://dev.mysql.com/doc/refman/8.0/en/show-columns.html)语句查看有关此表的列的其他信息，包括数据类型和默认值。

+   查看使用信息，请参阅第 17.9.1.4 节，“在运行时监视 InnoDB 表压缩”和第 17.15.1.3 节，“使用压缩信息模式表”。有关`InnoDB`表压缩的一般信息，请参阅第 17.9 节，“InnoDB 表和页压缩”。
