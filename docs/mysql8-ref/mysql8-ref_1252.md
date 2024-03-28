> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-examples-compression-sect.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-information-schema-examples-compression-sect.html)

#### 17.15.1.3 使用压缩信息模式表

**示例 17.1 使用压缩信息模式表**

以下是包含压缩表的数据库的示例输出（参见第 17.9 节，“InnoDB 表和页压缩”，`INNODB_CMP`，`INNODB_CMP_PER_INDEX`和`INNODB_CMPMEM`）。

以下表格显示了在轻量级工作负载下`INFORMATION_SCHEMA.INNODB_CMP`的内容。缓冲池中唯一包含的压缩页大小为 8K。自统计数据重置以来，压缩或解压页的时间不到一秒，因为`COMPRESS_TIME`和`UNCOMPRESS_TIME`列的值为零。

| 页大小 | 压缩操作 | 压缩操作成功 | 压缩时间 | 解压操作 | 解压时间 |
| --- | --- | --- | --- | --- | --- |
| 1024 | 0 | 0 | 0 | 0 | 0 |
| 2048 | 0 | 0 | 0 | 0 | 0 |
| 4096 | 0 | 0 | 0 | 0 | 0 |
| 8192 | 1048 | 921 | 0 | 61 | 0 |
| 16384 | 0 | 0 | 0 | 0 | 0 |

根据`INNODB_CMPMEM`，缓冲池中有 6169 个压缩的 8KB 页。唯一的其他分配块大小为 64 字节。`INNODB_CMPMEM`中最小的`PAGE_SIZE`用于那些在缓冲池中不存在未压缩页的压缩页的块描述符。我们看到有 5910 个这样的页。间接地，我们看到 259（6169-5910）个压缩页也以未压缩形式存在于缓冲池中。

下表显示了在轻负载工作负载下`INFORMATION_SCHEMA.INNODB_CMPMEM`的内容。由于压缩页内存分配器的碎片化，一些内存无法使用：`SUM(PAGE_SIZE*PAGES_FREE)=6784`。这是因为小内存分配请求通过从主缓冲池分配的 16K 块开始，使用伙伴分配系统来拆分更大的块来满足。碎片化很低是因为一些已分配的块已经被重定位（复制）以形成更大的相邻空闲块。这些复制的`SUM(PAGE_SIZE*RELOCATION_OPS)`字节消耗不到一秒的时间`(SUM(RELOCATION_TIME)=0)`。

| 页大小 | 已使用页数 | 空闲页数 | 重定位操作 | 重定位时间 |
| --- | --- | --- | --- | --- |
| 64 | 5910 | 0 | 2436 | 0 |
| 128 | 0 | 1 | 0 | 0 |
| 256 | 0 | 0 | 0 | 0 |
| 512 | 0 | 1 | 0 | 0 |
| 1024 | 0 | 0 | 0 | 0 |
| 2048 | 0 | 1 | 0 | 0 |
| 4096 | 0 | 1 | 0 | 0 |
| 8192 | 6169 | 0 | 5 | 0 |
| 16384 | 0 | 0 | 0 | 0 |
