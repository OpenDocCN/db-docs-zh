> 原文：[`dev.mysql.com/doc/refman/8.0/en/innodb-compression-background.html`](https://dev.mysql.com/doc/refman/8.0/en/innodb-compression-background.html)

#### 17.9.1.1 表压缩概述

因为处理器和缓存内存的速度增加比磁盘存储设备更快，许多工作负载是磁盘限制的。数据压缩可以使数据库大小更小，减少 I/O，并提高吞吐量，但会增加 CPU 利用率。压缩对于读密集型应用程序特别有价值，在具有足够 RAM 的系统上，可以将经常使用的数据保留在内存中。

使用`ROW_FORMAT=COMPRESSED`创建的`InnoDB`表可以在磁盘上使用比配置的`innodb_page_size`值更小的页面大小。较小的页面需要更少的 I/O 来从磁盘读取和写入，这对 SSD 设备特别有价值。

压缩页面大小是通过`CREATE TABLE`或`ALTER TABLE`的`KEY_BLOCK_SIZE`参数指定的。不同的页面大小要求表必须放置在 file-per-table 表空间或 general tablespace 中，而不是在 system tablespace 中，因为系统表空间无法存储压缩表。有关更多信息，请参见 Section 17.6.3.2, “File-Per-Table Tablespaces”和 Section 17.6.3.3, “General Tablespaces”。

无论`KEY_BLOCK_SIZE`值如何，压缩级别都是相同的。当您为`KEY_BLOCK_SIZE`指定较小的值时，您将获得越来越小页面的 I/O 优势。但是，如果指定的值太小，当数据值无法压缩到足以适应每个页面的多行时，重新组织页面会增加额外的开销。对于每个索引的关键列的长度，表的`KEY_BLOCK_SIZE`可以有一个硬限制。指定一个太小的值，`CREATE TABLE`或`ALTER TABLE`语句将失败。

在缓冲池中，压缩数据以小页面的形式保存，页面大小基于`KEY_BLOCK_SIZE`值。为了提取或更新列值，MySQL 还在缓冲池中创建一个包含未压缩数据的未压缩页面。在缓冲池中，对未压缩页面的任何更新也会重新写回等效的压缩页面。您可能需要调整缓冲池的大小以容纳压缩和未压缩页面的额外数据，尽管在需要空间时，未压缩页面会从缓冲池中被驱逐，然后在下一次访问时再次解压缩。
